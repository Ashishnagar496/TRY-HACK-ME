# valenfind

## **Objective**

Identify and exploit vulnerabilities in a dating web application to retrieve sensitive information (flag).

## **Reconnaissance**

- Registered a user account to explore the application's functionalities:
    - Register
    - Login
    - Profile update
    - View user profiles
    - Like user profiles
    - Send valentine to a user
- Noticed a user profile for "cupid" with the description:
    
    `"I keep the database secure. No peeking."`
    
    This hints at a focus on database security and suggests potential hidden functionality.
    

## **Vulnerability Discovery**

- Sent a request to `/cupid` and observed comments in the response:
    
    javascript
    
    ```
    // Feature: Dynamic Layout Fetching
    // Vulnerability: 'layout' parameter allows LFI
    fetch(`/api/fetch_layout?layout=${layoutName}`)
    ```
    
- Intercepted a request to `/api/fetch_layout?layout=theme_classic.html`.
- Changed the `layout` parameter to `cupid` and received an error revealing the server path:
    
    text
    
    ```
    Error loading theme layout: [Errno 2] No such file or directory: '/opt/Valenfind/templates/components/cupid'
    ```
    
- The error indicates the application is written in Python (as confirmed by response headers) and discloses the absolute path.

## **Local File Inclusion (LFI) Exploitation**

- Leveraged the LFI vulnerability to read source code files by traversing the directory structure:
    - `/opt/Valenfind/templates/components/app.py`
    - `/opt/Valenfind/templates/app.py`
    - `/opt/Valenfind/app.py`
- Successfully retrieved the Python source code, which contained sensitive information:
    - Admin API key
    - Database filename
    - A code snippet revealing an export endpoint:
        
        python
        
        ```
        @app.route('/api/admin/export_db')
        def export_db():
            auth_header = request.headers.get('X-Valentine-Token')
        
            if auth_header == ADMIN_API_KEY:
                try:
                    return send_file(DATABASE, as_attachment=True, download_name='valenfind_leak.db')
                except Exception as e:
                    return str(e)
        ```
        
- The source code also showed security filters that block requests containing `cupid.db`, any `.db` extension, or `seeder.py` to prevent direct access.

## **Database Extraction**

- Constructed a request to `/api/admin/export_db` with the required header:
    
    text
    
    ```
    GET /api/admin/export_db HTTP/1.1
    Host: <target>
    X-Valentine-Token: <admin_api_key_from_source>
    ```
    
- The server responded with the database file as an attachment.
- Upon opening the database (or analyzing the response), the flag was found.

## **Flag Obtained**

- The flag was retrieved from the database response (format: `THM{...}`)