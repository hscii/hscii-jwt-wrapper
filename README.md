# hscii-jwt-wrapper

Lightweight Rust wrapper for JWT authentication in HSCII microservices. Simplifies token generation, verification, and centralized auth management across distributed systems.

## Usage

1. Add the public git url to your service's Cargo.toml

   ```toml
   [dependencies]
   worker = "0.0.18"
   hscii_auth_jwt = { git = "https://github.com/hscii/hscii-jwt-wrapper.git" }
   serde_json = "1.0"
   ...
   ```

2. Intitialize the wrapper and use `impl HsciiAuthJwt`

   ```rust
   use worker::*;
   use hscii_jwt_wrapper;

   #[event(fetch)]
   pub async fn main(req: Request, env: Env, _ctx: worker::Context) -> Result<Response> {
       // Get the Authorization header
       let auth_header = req.headers().get("Authorization")?;

       // Extract the token (assuming Bearer token)
       let token = auth_header.strip_prefix("Bearer ").ok_or("Invalid Authorization header")?;

       // Initialize the HsciiAuthJwt
       let project_id = env.secret("FIREBASE_PROJECT_ID")?.to_string();
       let auth = hscii_jwt_wrapper::init(&project_id).await?;

       // Verify the token
       match auth.verify_token(token).await {
           Ok(claims) => {
               // Token is valid, handle authenticated request
               console_log!("Authenticated user: {}", claims.user_id);
           },
           Err(e) => {
               // Token is invalid, send error response
               console_error!("Authentication failed: {:?}", e);
               Response::error("Unauthorized", 401)
           }
       }
   }

   ...
   ```
