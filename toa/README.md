# Talking Objects Archive

### Website
- [Talking Objects Archive](https://talkingobjectsarchive.org/)

### Code 
- [Talkinb Object Archive & Server](https://github.com/talking-objects/talking_objects)

# Table of Contents

#### 1. Frontend
- [Next.js Application](#nextjs-application)
- [Components](#components)
- [Data Fetching](#data-fetching)

#### 2. Backend
- [Kirby CMS](#kirby-cms)
- [API Endpoints](#api-endpoints)
- [Content Structure](#content-structure)

# 1. Frontend

### Next.js Application
- Built with Next.js framework
- Server-side rendering for improved performance
- Static site generation for content pages
- Client-side navigation between pages

### Data Fetching
- API integration with Kirby backend
- Server-side data fetching
- Client-side data caching
- Dynamic content updates

### Headless KQL
- Kirby Query Language for data fetching
- Flexible content querying
- Structured data responses
- Custom query parameters
- Optimized performance

### API Communication Examples

#### 1. Base API Fetcher Function
This function handles the basic API communication with Kirby CMS using authentication.

```javascript
export const fetchDataOriginAPI = async ({bodyD, language="en"}) => {
    try {
      const api = `${process.env.KB_API}`;
      const username = `${process.env.KB_USERNAME}`;
      const password = `${process.env.KB_PASSWORD}`;
  
      const encodedAuthString = Buffer.from(`${username}:${password}`).toString("base64");
      const headerAuthString = `Basic ${encodedAuthString}`;
      const bodyData = bodyD
      const response = await fetch(api, {
        method: "POST",
        headers: {
          Authorization: headerAuthString,
          "Content-Type": "application/json",
          'X-Language': language
        },
        cache: "no-store",
        body: JSON.stringify(bodyData)
      });
  
      const dataKirby = await response.json();
      return dataKirby;
    } catch (error) {
        
    }
  };
   
export const DynamicInfoPageChildrenData = ({currentLanguage,slug}) => {
  const bodyData = {
    query: `page("info").children.filterBy('slug', '==', '${slug}')`,
    select: {
      slug: true,
      uuid: true, 
      info_title: true,
      info_header_text: true,
      contents: true,
    }
  }
  const { data, error, isLoading } = useSWR(
    // The key array will be passed to the fetcher as the argument
    { bodyD:bodyData, language:currentLanguage },  // This will be passed as the first argument to the fetcher
    fetchDataOriginAPI   // The fetcher function
  );

  return {
    dataSwr: data,
    error: error,
    isLoading: isLoading
}

}

```

**Parameters:**
- `bodyD`: Query body data for KQL
- `language`: Language code for content localization (default: "en")

#### 2. Dynamic Data Fetching with KQL and SWR
Example of fetching dynamic page content using Kirby Query Language (KQL) and SWR for data management.

```javascript
export const DynamicInfoPageChildrenData = ({currentLanguage, slug}) => {
  // ... KQL query and SWR implementation ...
}
```

**Parameters:**
- `currentLanguage`: Active language for content
- `slug`: Page identifier

**Returns:**
- `dataSwr`: Fetched data
- `error`: Error state
- `isLoading`: Loading state

### Usage Notes
- Uses environment variables for API configuration
- Implements basic authentication
- Includes caching control with `no-store`
- Supports multilingual content through `X-Language` header

# 2. Backend

### Kirby CMS
- PHP-based headless CMS
- Content management system
- File-based data structure
- Custom panel interface

### API Endpoints
- RESTful API architecture
- Content delivery endpoints
- Authentication handling
- Media file management

### CORS Configuration

This section handles Cross-Origin Resource Sharing (CORS) configuration for the backend API.

#### 1. Origin Configuration
```php
<?php
// Allowed origins
$allowedOrigins = [
    'https://talkingobjectsarchive.org',
    'https://www.talkingobjectsarchive.org',
    'https://staging.talkingobjectsarchive.org',
];

// Get the origin of the request
$origin = $_SERVER['HTTP_ORIGIN'] ?? '';

// Check if the origin is in the allowed list
if (in_array($origin, $allowedOrigins)) {
    $allowedOrigin = $origin;
} else {
    $allowedOrigin = ''; // Block requests from disallowed origins
}

// Handling Preflight Requests
if ($_SERVER['REQUEST_METHOD'] === 'OPTIONS') {
    if ($allowedOrigin) {
        header("Access-Control-Allow-Origin: $allowedOrigin");
        header("Access-Control-Allow-Methods: GET, POST, OPTIONS, PATCH, PUT");
        header("Access-Control-Allow-Headers: Content-Type, Authorization, x-language");
        header("Access-Control-Allow-Credentials: true");
    }
    http_response_code(200);
    exit(0);
}

// Setting Headers
if ($allowedOrigin) {
    header("Access-Control-Allow-Origin: $allowedOrigin");
    header("Access-Control-Allow-Methods: GET, POST, OPTIONS, PATCH, PUT");
    header("Access-Control-Allow-Headers: Content-Type, Authorization, x-language");
    header("Access-Control-Allow-Credentials: true");
}

```

**Purpose:**
- Defines whitelist of allowed domains
- Includes development, production, and staging environments
- Prevents unauthorized cross-origin requests

#### 2. Security Implementation
```php
<?php
// ... CORS implementation code ...
```

**Key Features:**
- Origin validation
- Preflight request handling
- Secure header configuration
- Request method control

**Supported Functionalities:**
- HTTP Methods: GET, POST, OPTIONS, PATCH, PUT
- Custom Headers: Content-Type, Authorization, x-language
- Credentials: Enabled for authenticated requests

### Usage Notes
1. **Development Setup**
   - Local development supported via `localhost:3000`
   - Staging environment configured for testing

2. **Security Considerations**
   - Strict origin checking
   - Blocked requests from unauthorized origins
   - Proper preflight request handling

3. **Headers Configuration**
   - CORS headers set only for allowed origins
   - Essential headers for API functionality
   - Support for multilingual content
