# ColdBox REST HMVC Template - AI Coding Instructions

This is a ColdBox template for building **modular RESTful APIs** using HMVC (Hierarchical Model-View-Controller) architecture with JWT authentication, OpenAPI documentation, and security. The template pre-configures an `api` module with a nested `v1` sub-module for API versioning.

## 🏗️ HMVC Architecture Overview

**Key Design Decision**: This template uses nested modules (`api/v1`) with **inherited entry points** to create URL structures like `/api/v1/resource`. Each module can contain its own handlers, models, routes, and configuration.

### Modular Structure

```
/modules_app/              - Application modules (HMVC)
└── api/                   - API parent module
    ├── ModuleConfig.cfc  - Module settings (entryPoint="api", inheritEntryPoint=true)
    ├── models/           - API-level models
    └── modules_app/      - Nested sub-modules
        └── v1/           - API version 1
            ├── ModuleConfig.cfc  - V1 settings (entryPoint="v1", inheritEntryPoint=true)
            ├── handlers/         - V1 REST handlers
            │   ├── Echo.cfc     - Example endpoint
            │   └── Auth.cfc     - Authentication
            └── config/
                └── Router.cfc    - V1 route definitions

/handlers/                 - Root-level handlers (Main.cfc)
/models/                   - Root-level models
/config/                   - Root configuration
    ├── ColdBox.cfc       - Framework settings
    └── Router.cfc        - Root routes
```

### Inherited Entry Points

**CRITICAL**: Entry points cascade through module hierarchy:

```cfml
// modules_app/api/ModuleConfig.cfc
this.entryPoint = "api";
this.inheritEntryPoint = true;  // Root becomes /api

// modules_app/api/modules_app/v1/ModuleConfig.cfc
this.entryPoint = "v1";
this.inheritEntryPoint = true;  // Inherits parent, becomes /api/v1
```

**Result**: A handler action `v1:Echo.index` is accessible at `/api/v1/echo` or via event: `event=v1:Echo.index`.

## 📝 REST Handler Patterns

All API handlers extend `coldbox.system.RestHandler`:

```cfml
component extends="coldbox.system.RestHandler" {

    // Dependency injection
    property name="userService" inject="UserService";

    // HTTP method security
    this.allowedMethods = {
        "index"  : "GET",
        "create" : "POST",
        "update" : "PUT,PATCH",
        "delete" : "DELETE"
    };

    /**
     * OpenAPI annotations for auto-documentation
     * @x        -route     (GET) /api/v1/echo
     * @response -default   ~api/v1/echo/index/responses.json##200
     */
    function index(event, rc, prc){
        event.getResponse().setData("Welcome to my API");
    }

    /**
     * Secured endpoint - requires authentication
     * @x        -route     (GET) /api/v1/whoami
     * @security bearerAuth,ApiKeyAuth
     * @response -default   ~api/v1/echo/whoami/responses.json##200
     * @response -401       ~api/v1/echo/whoami/responses.json##401
     */
    function whoami(event, rc, prc) secured {
        event.getResponse().setData(
            jwtAuth().getUser().getMemento()
        );
    }
}
```

### RestHandler Built-in Constants

```cfml
// HTTP Methods
METHODS.GET, METHODS.POST, METHODS.PUT, METHODS.PATCH, METHODS.DELETE, METHODS.HEAD

// HTTP Status Codes
STATUS.SUCCESS (200), STATUS.CREATED (201), STATUS.NO_CONTENT (204)
STATUS.BAD_REQUEST (400), STATUS.NOT_AUTHORIZED (401), STATUS.NOT_FOUND (404)
STATUS.EXPECTATION_FAILED (417), STATUS.INTERNAL_ERROR (500)
```

### RestHandler Built-in Error Handling

```cfml
// Automatically catches exceptions
function onError(event, rc, prc){
    var exception = prc.exception;
    event.getResponse()
        .setError(true)
        .setStatusCode(STATUS.INTERNAL_ERROR)
        .setStatusText("Error")
        .addMessage(exception.message);
}

// Handles invalid HTTP methods automatically
function onInvalidHTTPMethod(event, rc, prc){
    event.getResponse()
        .setError(true)
        .setStatusCode(STATUS.NOT_ALLOWED)
        .addMessage("Invalid HTTP Method");
}
```

## 🗺️ Modular Routing

### Module-Level Routes (v1/config/Router.cfc)

```cfml
component {
    function configure(){
        // Routes are scoped to module entry point (/api/v1)
        get("/", "Echo.index");                    // GET /api/v1/
        
        // Authentication routes
        post("/login", "Auth.login");              // POST /api/v1/login
        post("/register", "Auth.register");        // POST /api/v1/register
        post("/logout", "Auth.logout");            // POST /api/v1/logout
        
        // Secured routes
        get("/whoami", "Echo.whoami");            // GET /api/v1/whoami
        
        // RESTful resources
        resources("users");                        // Full CRUD for /api/v1/users
        
        // Conventions-based catch-all
        route("/:handler/:action").end();
    }
}
```

**CRITICAL**: Routes in module config are **relative to the module's inherited entry point**.

## 🔐 Authentication (CBSecurity + JWT)

### Login Flow (Auth.cfc)

```cfml
function login(event, rc, prc){
    param rc.username = "";
    param rc.password = "";
    
    // jwtAuth() provided by CBSecurity
    // Throws InvalidCredentials exception on failure (caught by RestHandler)
    var token = jwtAuth().attempt(rc.username, rc.password);
    
    event.getResponse()
        .setData(token)
        .addMessage("Bearer token created, expires in X minutes");
}
```

### Accessing Authenticated User

```cfml
function whoami(event, rc, prc) secured {
    // Get authenticated user
    var user = jwtAuth().getUser();
    
    // Return user data as memento
    event.getResponse().setData(user.getMemento());
}
```

### Security Annotations

- **`secured`** modifier on function - Requires authentication
- **`@security bearerAuth`** - Documents auth requirement for OpenAPI

## 💉 Dependency Injection (WireBox)

Models are namespaced by module:

```cfml
// In v1 module handler
property name="userService" inject="UserService";     // From v1 models/
property name="apiService" inject="api:SomeService";  // From api module models/
property name="rootService" inject="SomeService";     // From root models/

// Cross-module injection
property name="v2Service" inject="v2:SomeService";    // From v2 module (if exists)
```

**Auto-Discovery**: When `this.autoMapModels = true` in `ModuleConfig.cfc`, all CFCs in module's `models/` are registered with module namespace.

## 🛠️ Build Commands

```bash
# Install dependencies (coldbox, cbsecurity, mementifier, cbvalidation)
box install

# Start server
box server start

# Code formatting
box run-script format              # Format all CFML
box run-script format:check        # Check formatting
box run-script format:watch        # Watch and auto-format

# Testing
box testbox run                    # Run all tests

# Docker
box run-script docker:build        # Build image
box run-script docker:run          # Run container
box run-script docker:stack up     # Start compose stack
```

## 📚 Key Dependencies

From `box.json`:

- **cbsecurity** (^3.0.0) - JWT authentication, authorization rules
- **mementifier** (^3.3.0) - Entity serialization (getMemento() method)
- **cbvalidation** (^4.1.0) - Model validation (validateOrFail())
- **relax** (dev) - REST API documentation visualizer
- **route-visualizer** (dev) - Visual route debugger

## 🎯 Configuration Patterns

### Default Event (config/ColdBox.cfc)

```cfml
variables.coldbox = {
    // Default event points to module handler
    defaultEvent: "v1:Echo.index",
    
    // Exception handler in module
    exceptionHandler: "v1:Echo.onError",
    
    // Module locations
    modulesExternalLocation: []
};
```

### Module Configuration (ModuleConfig.cfc)

```cfml
component {
    this.title = "v1";
    this.entryPoint = "v1";
    this.inheritEntryPoint = true;  // CRITICAL for URL structure
    this.modelNamespace = "v1";     // Namespace for DI
    this.cfmapping = "v1";          // CF mapping
    this.autoMapModels = true;      // Auto-register models
    
    function configure(){
        // Module settings
        settings = {};
        
        // Module routes defined in config/Router.cfc
    }
}
```

## 🚨 Common Pitfalls

1. **Entry Point Inheritance**: Forgetting `this.inheritEntryPoint = true` breaks URL structure
2. **Module Namespacing**: Handlers must be called with `v1:Handler.action` syntax
3. **RestHandler Required**: Don't extend `EventHandler` - must extend `RestHandler`
4. **Secured Modifier**: Use `secured` keyword on function, not annotation
5. **JWT Configuration**: CBSecurity JWT settings in `config/modules/cbsecurity.cfc`
6. **Model Namespaces**: Injection must include module namespace for module models
7. **Route Relativity**: Routes in module Router.cfc are relative to entry point

## 📖 Module Event Syntax

```cfml
// In config or handlers, reference module events:
"v1:Echo.index"         // Handler in v1 module
"api:SomeHandler.save"  // Handler in api module (parent)
"Main.index"            // Root handler (no module)

// In views/execute calls:
event.execute("v1:Echo.index")
relocate("v1:Echo.index")
```

## 🔍 Debugging

```cfml
// Enable debug in config/ColdBox.cfc
variables.settings = { debugMode: true };

// Log in handlers
property name="log" inject="logbox:logger:{this}";
log.info("Debug message", {data: rc});

// RestHandler response debugging
event.getResponse()
    .setData(someData)
    .setDebugMode(true)  // Includes stack traces in response
    .addMessage("Debug info");
```

## 📚 Key Files

- `modules_app/api/ModuleConfig.cfc` - API module configuration
- `modules_app/api/modules_app/v1/ModuleConfig.cfc` - V1 module configuration
- `modules_app/api/modules_app/v1/config/Router.cfc` - V1 routes
- `modules_app/api/modules_app/v1/handlers/` - V1 REST handlers
- `config/ColdBox.cfc` - Root config (defaultEvent, exceptionHandler)
- `box.json` - Dependencies (cbsecurity, mementifier, cbvalidation)

## 📖 Documentation

- ColdBox HMVC: https://coldbox.ortusbooks.com/hmvc/modules
- CBSecurity: https://coldbox-security.ortusbooks.com
- RestHandler: https://coldbox.ortusbooks.com/the-basics/event-handlers/rest-handler
- Mementifier: https://forgebox.io/view/mementifier
- CBValidation: https://coldbox-validation.ortusbooks.com
