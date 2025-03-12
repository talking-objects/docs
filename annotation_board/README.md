# EVA Project Documentation

### Website
- [EVA](https://eva.talkingobjectsarchive.org/)
- [Annotation Board](https://board.talkingobjectsarchive.org/)
- [Annotation Board Admin](https://admin.talkingobjectsarchive.org/admin/)

### Code 
- [Annotation_Board & Annotation_Board_Server](https://github.com/talking-objects/annotation_board)
- [EVA](https://github.com/talking-objects/archive/tree/main/frontend/frontend-nextjs)

# Table of Contents
#### 1. Frontend
- [Annotation Board](#annotation-board)
- [EVA Client](#eva-client) 
- [Data Fetching](#data-fetching)

#### 2. Backend
- [Admin](#admin)
  - [Structure](#structure)
  - [Video](#video)
  - [Annotation](#annotationcategory-place-event-reference-data-tag-narration)
  - [Edit Videos](#edit_videos--deprecated)
- [Pagination](#pagination)

#### 3. API Reference
- [Authentication](#authentification)
- [Videos](#videos)
- [Clips](#clips)

#### 4. Database Schema
- [Model Relationships](#model-relationships)
- [Users](#users)
- [Videos](#videos)
- [Annotation Wrapper](#annotation-wrapper)
- [Annotations](#annotations)
- [Clips](#clips)

#### 5. Security
- [CSRF Protection](#csrf-protection)
- [CORS Protection](#cors-protection)
- [Session Authentication](#session-authentication)
- [API Key Authentication](#api-key-authentication)
- [OAuth 2.0 Authentication](#oauth-20-authentication)

#### 6. Deployment
- [Docker](#docker)
- [Hetzner](#hetzner)

#### 7. Annotation Board Documentation
- [Docs](./annotation.md)

#### 8. Annotation Board Admin Documentation
- [Docs](./admin.md)



## 1. Frontend

### Annotation Board
- A web application for video annotation and analysis
- Built with modern web technologies:
  - Next.js and React for the frontend framework
  - Tailwind CSS for styling
  - React Query for state management
  - Axios for API communication

### EVA Client
- A web application that serves as the main interface for the EVA project
- Built with modern web technologies:
  - Next.js and React for the frontend framework
  - Tailwind CSS for styling
  - React Query for state management
  - Axios for API communication

### Data Fetching
```shell
- UseQuery
    - Axios
        - Credential : true
```
Using useQuery for Fetching and Maintaining User Authentication State
When fetching user data in Next.js with Axios and React Query, it's essential to ensure that authentication credentials (such as session cookies) are properly handled. This allows to maintain the login state across different requests.

```jsx
const instance = axios.create({
    baseURL: process.env.BASE_URL,
    withCredentials: true
})

export const getMe = () =>
    instance.get(`users/me`).then((response) => response.data);
```
- custom hook
- Instead of manually calling getMe(), I wrap it inside useQuery to manage state efficiently:
```js
import { getMe } from "@/app/api"
import { useQuery } from "@tanstack/react-query"

export const useUser = () => {
    const {isLoading, data, isError} = useQuery({
        queryKey: ["me"],
        queryFn: getMe
    });

    return {
        userLoading: isLoading,
        user: data,
        isLoggedIn: !isError
    }
}
```
--- 

## 2. Backend
### Technology Stack
- RESTful API server built with Django
- API implementation using Django Rest Framework (DRF)
- Database management through Django ORM
- Administrative interface with Django Admin
- Session-based authentication system
- Access control based on HTTP methods

Key Features:
- Follows RESTful API architecture
- Session-based user authentication
- Data management through admin dashboard
- Efficient database operations using ORM


### Admin
#### URL
```shell
/admin
```
#### Structure
```shell
- Users # Only my user model
- Videos # Show all
    - Annotation Wrapper # Hide
        - Reference(Annotation) # Show all
        - Tag(Annotation) # Show all
        - Place(Annotation) # Show all
        - Category(Annotation) # Show all
        - Event(Annotation) # Show all
        - Data(Annotation) # Show all
- Clips # Hide
- EditVideos # Show all
    - EditVideo Wrapper # Hide
        - Videos # Show all
```

#### Video
```py
class VideoAdmin(admin.ModelAdmin):
    list_display = (
        "title",
        "user",
        "author",
        "contributors",
        "genre",
        "place",
        "country",
        "language",
    )
    list_filter = (
        "user",
        "author",
        "contributors",
        "genre",
        "place",
        "country",
        "language",
    )
    search_fields = (
        "title",
        "description",
        "author",
        "contributors",
        "genre",
        "place",
        "country",
        "language",
    )
```

#### Annotation(Category, Place, Event, Reference, Data, Tag, Narration)
```py
class DataAdmin(admin.ModelAdmin):
    fields = ("start", "end", "annotation_wrapper", "value")
    list_display = ("__str__", "start", "end", "type")
    list_filter = ("annotation_wrapper__video__title", "start", "end")
    search_fields = ("annotation_wrapper__video__title", "start", "end")

    def has_add_permission(self, request):
        return False
```


### Pagination
- Using Django Rest Framework's built-in PageNumberPagination
- Implements page-based pagination with customizable page size
- Returns total pages and total count in response

👨🏻‍💻(Dain:) If we have a lot of data in the future, then I will use __Limitoffset__ pagination.

```py
class ClipView(APIView):
    def get(self, request):
        random = request.query_params.get("random", "false")
        all_clips = Clip.objects.all()

        if random == "true":
            all_clips = all_clips.order_by("?")
        else:
            all_clips = all_clips.order_by("-created")

        # pagination
        page_limit = request.query_params.get("page_limit", "10")
        page = request.query_params.get("page", "1")

        try:
            page = int(page)
            if page < 1:
                page = 1
        except ValueError:
            page = 1

        if page_limit:
            try:
                page_limit = int(page_limit)
                if page_limit < 1:
                    page_limit = 1
            except ValueError:
                page_limit = 10

        paginator = PageNumberPagination()
        paginator.page_size = int(page_limit)
        paginator.page = int(page)
        total_pages = (all_clips.count() + page_limit - 1) // page_limit
        total_clips_count = all_clips.count()

        try:
            paginated_clips = paginator.paginate_queryset(all_clips, request)
            if paginated_clips is None:
                return Response(
                    {
                        "data": [],
                        "total_pages": total_pages,
                        "total_clips_count": total_clips_count,
                    }
                )
            else:
                all_clips = paginated_clips
        except Exception:
            return Response(
                {
                    "data": [],
                    "total_pages": total_pages,
                    "total_clips_count": total_clips_count,
                }
            )

        serializer = ClipSerializer(all_clips, many=True)
        return Response(
            {
                "data": serializer.data,
                "total_pages": total_pages,
                "total_clips_count": total_clips_count,
            }
        )
```

### Serialization
- __Using Serializer of Django_Rest_framework(DRF)__
- ModelSerializer

### ORM
- __Using Django ORM__


### Authentification
#### Rest Framework Simple JWT vs PyJWT vs Django Authentification
- [Django Session](https://docs.djangoproject.com/en/5.1/topics/auth/)
- [SimpleJWT](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/getting_started.html)
- [PyJWT](https://pyjwt.readthedocs.io/en/stable/)
- We are using Django Session now. 


### CORS & CSRF
- CSRF=["client donmain"]
- CORS=["client donmain"]

### GraphQL (Nice to have) ⭐️

## 3. API Reference
- Using DRF(Djagno_Rest_Framework) - APIView


### Authentification
```shell
# get my user data
GET /api/v1/users/me

# update my user data
PUT /api/v1/users/me 🚧 (deprecated) 👨🏻‍💻 You can update your data on the admin page

# create a user
POST /api/v1/users

# Log-In
POST /api/v1/users/log-in

# Log-Out
POST /api/v1/users/log-out

# Change Password
POST /api/v1/users/change-password 🚧 (deprecated) 👨🏻‍💻 You can update your data on the admin page
```

### Videos
```shell
# Get all videos
GET /api/v1/videos 
    - random: bool (default: false)
    - page: int (default: 1) # pagination
    - page_limit: int (default: 10) # pagination

    - Request:  
    curl -X GET "http://localhost:8000/api/v1/videos?random=true&page=1&page_limit=10"
    
    - Response:
    {
        "total_pages": 10,  # total number of pages
        "total_videos_count": 100,  # total number of videos
        "data": [
            {
                "pk": 1,
                "title": "Sample Video 1",
                "description": "This is a sample video",
                "url": "https://example.com/video1.mp4",
                "thumbnail": "https://example.com/thumbnail1.jpg",
                "duration": 180,  # in seconds
                "created_at": "2024-01-01T00:00:00Z",
                "updated_at": "2024-01-01T00:00:00Z",
                //... more fields
            },
            // ... more videos
        ]
    }   

# Get a video
GET /api/v1/videos/<int:pk>

    - Request:
    curl -X GET "http://localhost:8000/api/v1/videos/1"

    - Response:
    {
        "pk": 1,
        "title": "Sample Video 1",
        "description": "This is a sample video",
        // more fields...
    }

# Search videos
GET /api/v1/videos/search
    - query: str (default: "")  # search query (title, description, author, contributors, genre, place, country, language)
    - page: int (default: 1) # pagination
    - page_limit: int (default: 10) # pagination

    - Request:
    curl -X GET "http://localhost:8000/api/v1/videos/search?query=sample"
    
    - Response:
    {
        "total_pages": 10,  # total number of pages
        "total_videos_count": 100,  # total number of videos
    "data": [
        //... more videos
    ]
    }

# Create a video
POST /api/v1/videos
    
```

### Clips
- The Clips model will be generated when annotations are created and saved.
- So you don't need to generate.
```shell
# Get all clips
GET /api/v1/clips
    - random: bool (default: false)
    - page: int (default: 1) # pagination
    - page_limit: int (default: 10) # pagination

# Search clips
GET /api/v1/clips/search
    - query: str (default: "")  # search query (title, description, author, contributors, genre, place, country, language)
    - page: int (default: 1) # pagination
    - page_limit: int (default: 10) # pagination

# Get a clip
GET /api/v1/clips/<int:pk>
```

#### Edit_Videos 🚧 (deprecated)
```shell
# Get all edit videos
GET /api/v1/editvideos

# Get a edit video
GET /api/v1/editvideos/<int:pk>

# Create a edit video
POST /api/v1/editvideos
```

## 4. Database Schema

### Model Relationships
```shell
Users ----< Videos # One to Many(Foreign Key) onDelete=SET_NULL
```
```shell
Users ----< EditVideos # One to Many(Foreign Key) 🚧 (deprecated)
```
```shell
Videos ----- Annotation Wrapper # One to One onDelete=CASCADE
```
```shell
Annotation Wrapper ----< Annotation # One to Many(Foreign Key) onDelete=CASCADE
```
```shell
Clips ----- Annotation # One to One onDelete=CASCADE
```
```shell
EditVideos ----< EditVideo Wrapper # One to Many(Foreign Key) 🚧 (deprecated)
```
```shell
EditVideo Wrapper ---< Videos # One to Many(Foreign Key) 🚧 (deprecated)
```

### Users
| Field | Type | Description |
|-------|------|-------------|
| id | Integer | Primary key |
| username | String | Unique username |
| email | String | User's email address |
| password | String | Hashed password |
| is_active | Boolean | Account status |
| date_joined | DateTime | Account creation date |

### Videos
| Field | Type | Description |
|-------|------|-------------|
| id | Integer | Primary key |
| title | String | Video title |
| description | Text | Video description |
| user | ForeignKey | Reference to Users |
| author | String | Original content creator |
| contributors | String | Additional contributors |
| genre | String | Video genre |
| place | String | Recording location |
| country | String | Country of origin |
| language | String | Primary language |
| created_at | DateTime | Creation timestamp |
| updated_at | DateTime | Last update timestamp |

### Annotation_Wrapper
| Field | Type | Description |
|-------|------|-------------|
| id | Integer | Primary key |
| video | OneToOne | Reference to Videos |

### Annotations (Base)
| Field | Type | Description |
|-------|------|-------------|
| id | Integer | Primary key |
| start | Float | Start time in seconds |
| end | Float | End time in seconds |
| annotation_wrapper | ForeignKey | Reference to Annotation_Wrapper |
| type | String | Annotation type (Category/Place/Event/Reference/Data/Tag) |

### Clips
| Field | Type | Description |
|-------|------|-------------|
| id | Integer | Primary key |
| video | ForeignKey | Reference to Videos |
| annotation | OneToOne | Reference to Annotations |
| start_time | Float | Clip start time |
| end_time | Float | Clip end time |
| created_at | DateTime | Creation timestamp |



## 5. Security

### CORS Protection
- Implements `django-cors-headers` middleware
- Whitelist allowed origins in `CORS_ALLOWED_ORIGINS`
- Configure allowed HTTP methods
- Enable credentials for cookie handling
- Configuration example:
```python
CORS_ALLOWED_ORIGINS = [
    "https://your-frontend-domain.com",
    "http://localhost:3000"
]
CORS_ALLOW_CREDENTIALS = True  # Allow credentials
```

### CSRF Protection
- Uses Django's built-in CSRF middleware
- CSRF token required for all POST, PUT, DELETE requests
- Configured trusted domains in `CSRF_TRUSTED_ORIGINS`
- CSRF token delivered via cookies
- Implementation example:
```python
CSRF_TRUSTED_ORIGINS = [
    "https://your-frontend-domain.com",
    "http://localhost:3000"
]
CSRF_COOKIE_SECURE = True
CSRF_COOKIE_HTTPONLY = DEBUG == True
```

### Session Authentication
- Uses Django's default session-based authentication
- Session data stored in server-side database
- Session ID transmitted via cookies
- Configurable session expiry
- Key settings:
```python
SESSION_COOKIE_AGE = 1209600  # 2 weeks in seconds
SESSION_COOKIE_SECURE = True  # HTTPS only
SESSION_COOKIE_HTTPONLY = False
```

## 6. Deployment

### Docker
- Multi-stage builds for optimized images
- Separate containers for frontend and backend services
- Connected via Docker networks
- Environment variable configuration
- Example structure:
```bash
├── docker-compose.yml
    Services
    ├── frontend(Annotation Board)
    │   └── Dockerfile
    ├── frontend(EVA)
    │   └── Dockerfile
    ├── db(Postgresql)
    │   └── Dockerfile
    └── backend(Django)
        └── Dockerfile
    Networks
    └── External Network
    Volumes
    └── VolumeName
```




### Hetzner
- Cloud server hosting platform
- Firewall
- Backup
