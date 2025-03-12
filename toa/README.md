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

#### 4. Performance Optimizations

##### Image Optimization
- Next.js Image component with Sharp
  ```javascript
  // Example of optimized image usage
  import Image from 'next/image'
  
  <Image
    src="/path/to/image.jpg"
    width={800}
    height={600}
    alt="Optimized image"
    quality={75}
    placeholder="blur"
  />
  ```
- Features:
  - Automatic WebP/AVIF conversion
  - Responsive sizes
  - Lazy loading
  - Blur placeholder support
  - Quality optimization

##### Additional Optimizations
- Code splitting and lazy loading
  - Dynamic imports for components
  - Route-based code splitting
- Cache management strategies
  - SWR caching
  - Static page generation
- Performance monitoring
  - Lighthouse metrics
  - Core Web Vitals tracking

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

### Custom Plugins

#### 1. Category Management Plugin
Handles dynamic category extraction and management.

```php
Kirby::plugin('custom/method', [
    'pageMethods' => [
        'pluckCategoryNames' => function () {
            // Extracts unique category names
        },
        'pluckCategoryBoxes' => function () {
            // Extracts unique category tags
        }
    ]
]);
```

**Key Features:**
- `pluckCategoryNames`: Extracts unique category names from dynamic categories
- `pluckCategoryBoxes`: Extracts and flattens category tags from all pages
- Handles nested structure data
- Removes duplicates automatically

**Usage Example:**
```php
// In templates or controllers
$page->pluckCategoryNames();  // Returns array of unique category names
$page->pluckCategoryBoxes();  // Returns array of unique category tags
```

#### 2. Glossary Validation Plugin
Ensures data integrity for glossary entries.

```php
Kirby::plugin('custom/glossary-validation', [
    'hooks' => [
        'page.update:before' => function ($page, $values, $strings) {
            // Validates glossary entries
        }
    ]
]);
```

**Features:**
- Prevents duplicate glossary terms
- Case-insensitive validation
- YAML data structure support
- Automatic whitespace trimming

**Validation Rules:**
1. Checks for empty/null values
2. Converts YAML to array if needed
3. Case-insensitive duplicate checking
4. Whitespace normalization

**Error Handling:**
```php
throw new Exception('The term "' . $wordName . '" already exists...');
```

#### Implementation Notes
1. **Data Structure**
   - Uses Kirby's structure field type
   - Supports nested data
   - Handles multiple languages

2. **Performance**
   - Efficient array operations
   - Minimal database queries
   - Optimized for large datasets

3. **Maintenance**
   - Modular design for easy updates
   - Clear error messages
   - Documented validation rules
