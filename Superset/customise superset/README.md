Customizing the Superset UI allows you to tailor the appearance and functionality of your Superset instance to better fit your organization's branding and requirements. Here’s a detailed guide on how to customize various aspects of the Superset UI, including changing the logo and modifying row limits.

### Customizing the Superset Logo

1. **Locate the Logo Files:**
   - The Superset logo files are typically located in the `static/assets/images` directory of your Superset installation.

2. **Replace the Logo Files:**
   - Replace the default logo files (`superset-logo-horiz.png`, `superset-logo-horiz.svg`, `superset-logo-square.png`, etc.) with your custom logo files.
   - Ensure your custom logo files have the same names and dimensions as the default files.

3. **Update the Configuration:**
   - In your `superset_config.py` file, update the logo paths if you are using custom paths.
   - Example:
     ```python
     APP_ICON = '/static/assets/images/your-logo.png'
     ```

4. **Restart Superset:**
   - After making these changes, restart your Superset server to apply the new logo.

### Changing Row Limits

1. **Update the Configuration:**
   - Open your `superset_config.py` file and set the desired row limits.
   - Example:
     ```python
     ROW_LIMIT = 10000
     SQLLAB_CTAS_NO_LIMIT = True
     ```

2. **Apply the Changes:**
   - Save the configuration file and restart your Superset server for the changes to take effect.

### Customizing the UI with CSS

1. **Locate the CSS File:**
   - The CSS files for Superset are typically located in the `static/assets/stylesheets` directory.

2. **Create a Custom CSS File:**
   - Create a new CSS file or modify the existing CSS file to customize the appearance of Superset.
   - Example (custom.css):
     ```css
     body {
       background-color: #f0f0f0;
     }
     .navbar-brand {
       font-size: 24px;
       color: #333;
     }
     ```

3. **Update the Configuration:**
   - In your `superset_config.py` file, add the path to your custom CSS file.
   - Example:
     ```python
     EXTRA_CSS = ['/static/assets/stylesheets/custom.css']
     ```

4. **Apply the Changes:**
   - Save the configuration file and restart your Superset server for the changes to take effect.

### Customizing the UI with JavaScript

1. **Locate the JavaScript File:**
   - The JavaScript files for Superset are typically located in the `static/assets/javascripts` directory.

2. **Create a Custom JavaScript File:**
   - Create a new JavaScript file or modify the existing JavaScript file to customize the behavior of Superset.
   - Example (custom.js):
     ```javascript
     document.addEventListener('DOMContentLoaded', function() {
       console.log('Custom JavaScript loaded!');
     });
     ```

3. **Update the Configuration:**
   - In your `superset_config.py` file, add the path to your custom JavaScript file.
   - Example:
     ```python
     EXTRA_JS = ['/static/assets/javascripts/custom.js']
     ```

4. **Apply the Changes:**
   - Save the configuration file and restart your Superset server for the changes to take effect.

### Example of a Custom Superset Configuration (`superset_config.py`)

```python
# Custom logo
APP_ICON = '/static/assets/images/your-logo.png'

# Custom row limits
ROW_LIMIT = 10000
SQLLAB_CTAS_NO_LIMIT = True

# Custom CSS and JavaScript
EXTRA_CSS = ['/static/assets/stylesheets/custom.css']
EXTRA_JS = ['/static/assets/javascripts/custom.js']

# Other custom configurations
# Enable or disable features, set timeouts, configure databases, etc.
```

### Advanced Customization with Flask Templates

For more advanced UI customizations, you can modify the Flask templates used by Superset:

1. **Locate the Templates:**
   - The templates are located in the `superset/templates` directory.

2. **Modify the Templates:**
   - Edit the HTML and Jinja2 templates to change the structure and content of the pages.

3. **Apply the Changes:**
   - Save your changes and restart the Superset server.

### Example of Modifying a Flask Template

1. **Edit the Header Template:**
   - Open `superset/templates/appbuilder/baselist.html` and modify the header section.
   - Example:
     ```html
     {% block header %}
     <nav class="navbar navbar-default navbar-fixed-top">
       <div class="container-fluid">
         <div class="navbar-header">
           <a class="navbar-brand" href="/">
             <img alt="Brand" src="{{ url_for('static', filename='assets/images/your-logo.png') }}">
           </a>
         </div>
       </div>
     </nav>
     {% endblock %}
     ```

### Conclusion
Customizing the Superset UI allows you to create a tailored and branded experience for your users. By replacing logos, adjusting row limits, and adding custom CSS and JavaScript, you can significantly enhance the look and feel of your Superset instance. For more advanced customizations, modifying Flask templates provides deeper control over the UI. Always remember to restart your Superset server after making changes to apply the customizations.

### USING DOCKER COMPOSE

When using a Docker Compose installation for Apache Superset, you can customize the UI and other aspects of the application by modifying the relevant configuration files and mounting them into the Docker containers. Here’s a detailed guide on how to achieve this:

### Prerequisites

Ensure you have Docker and Docker Compose installed and that you have a working Docker Compose setup for Apache Superset.

### Step-by-Step Guide

#### 1. Locate Your Docker Compose File

Your Docker Compose file (`docker-compose.yml`) defines the services, volumes, and networks for your Superset setup. Typically, it looks something like this:

```yaml
version: '3.1'
services:
  superset:
    image: apache/superset
    container_name: superset
    ports:
      - "8088:8088"
    volumes:
      - ./superset_config:/app/pythonpath_dev
    environment:
      SUPERSET_CONFIG_PATH: /app/pythonpath_dev/superset_config.py
```

#### 2. Create the Custom Configuration Files

##### Create Custom CSS and JavaScript Files

Create a directory on your host machine to hold your custom files:

```bash
mkdir -p ./superset_custom/assets/stylesheets
mkdir -p ./superset_custom/assets/javascripts
```

Create your custom CSS and JavaScript files:

```css
/* superset_custom/assets/stylesheets/custom.css */
body {
  background-color: #f0f0f0;
}
.navbar-brand {
  font-size: 24px;
  color: #333;
}
```

```javascript
// superset_custom/assets/javascripts/custom.js
document.addEventListener('DOMContentLoaded', function() {
  console.log('Custom JavaScript loaded!');
});
```

##### Create Custom Superset Configuration

Create a custom `superset_config.py` file to include your custom CSS and JS files and other configurations:

```python
# superset_custom/superset_config.py

# Custom logo
APP_ICON = '/static/assets/images/your-logo.png'

# Custom row limits
ROW_LIMIT = 10000
SQLLAB_CTAS_NO_LIMIT = True

# Custom CSS and JavaScript
EXTRA_CSS = ['/static/assets/stylesheets/custom.css']
EXTRA_JS = ['/static/assets/javascripts/custom.js']

# Other custom configurations
# Enable or disable features, set timeouts, configure databases, etc.
```

##### Replace the Logo

Create a directory for your custom images and place your custom logo there:

```bash
mkdir -p ./superset_custom/assets/images
cp your-logo.png ./superset_custom/assets/images/
```

#### 3. Modify the Docker Compose File

Modify your `docker-compose.yml` file to mount the custom directories and configuration file:

```yaml
version: '3.1'
services:
  superset:
    image: apache/superset
    container_name: superset
    ports:
      - "8088:8088"
    volumes:
      - ./superset_custom:/app/superset_custom
    environment:
      SUPERSET_CONFIG_PATH: /app/superset_custom/superset_config.py
    command: >
      sh -c 'cp -r /app/superset_custom/* /app/pythonpath_dev && 
             /usr/bin/docker-entrypoint.sh'
```

This configuration mounts the `superset_custom` directory into the container and copies its contents to the appropriate locations inside the container.

#### 4. Build and Start the Containers

Build and start your containers using Docker Compose:

```bash
docker-compose up --build -d
```

This command builds the Docker images, incorporating your custom configurations and assets, and starts the containers.

### Additional Customization

For more advanced customizations, such as modifying Flask templates, you would need to:

1. **Locate the Template Files:**
   - The templates are typically located in the `superset/templates` directory inside the container.

2. **Create Custom Template Files:**
   - Create a custom templates directory and place your modified templates there.

3. **Mount the Custom Templates:**
   - Mount your custom templates directory in the `docker-compose.yml` file similarly to how you mounted the custom CSS and JS files.

### Conclusion

By following these steps, you can customize the UI of Apache Superset when using Docker Compose. This approach allows you to tailor the appearance and functionality of Superset to better fit your organization’s needs, including changing the logo, modifying row limits, and adding custom CSS and JavaScript.






