# Annotation Board Admin Documentation

## Table of Contents
1. [Introduction](#introduction)
2. [Features](#features)
   - [Key Management](#key-management)
   - [Video Management](#video-management)
   - [Annotation Management](#annotation-management)
   - [Clip Management](#clip-management)
3. [User Management](#user-management)
4. [Update Annotations](#update-annotations)

## Introduction
The Annotation Board admin interface provides administrative capabilities for the annotation system.

## Features

### Login Admin
- Access the admin interface at `/admin` endpoint
- [Admin](https://admin.talkingobjectsarchive.org/admin/)

### Account Creation
- Create an account on the annotation board website. [Annotation Board Docs](./annotation.md)
- A secret key is required for account creation
- Contact the TalkingObject Team to obtain the secret key


### Key Management
- TOA_KEY: Generate and manage registration keys for Create an account on the Annotation Board website 
- Create and distribute secret keys required for new user registration
- Monitor key usage and validity

![Admin Video](../images/eva9.webp)


### My Video Management
- View video list
- Edit video information
- Delete videos
- Manage video metadata
- You can find the videos that you created on the annotation board website

![Admin Video](../images/eva6.webp)
![Admin Video](../images/eva7.webp)

### My Annotation Management
- View annotation list
- Manage annotations by type
  - Categories
  - Places
  - Events
  - References
  - Data
  - Tags
  - Narrations
- Edit/Delete annotations

![Admin Video](../images/eva8.webp)

### Clip Management(Only Superuser)
- View clip list
- Manage clip information
- Delete clips

### User Management
- View user list(Only Superuser)
- Manage user permissions(Only Superuser)
- Edit user information


## Update Annotations


### Reference
- start: number(Float type)
- end: number(Float type)
- value: 
```json
{"value":
     {
      "text": "reference text", // Update Text here
      "url": "https://example.com" // Update URL here
    }
}
```

### Tag
- start: number(Float type)
- end: number(Float type)
- value: 
```json
{"value": "movie, drama, sport"} // Update value 
// Result 
// #movie #drama #sport
```

### Place
- start: number(Float type)
- end: number(Float type)
- value: 
```json
{"value": {"url": "https://example.com", "placeName": "Berlin", "text": "Vis", "latitude": "32", "longitude": "22"}}
```

### Narration
- start: number(Float type)
- end: number(Float type)
- value: 
```json
{"value": "Maam Njaré, the tutelary genius of the sea among the Lebu people of Yoff in Dakar."}
```

### Event
- start: number(Float type)
- end: number(Float type)
- value: 
```json
{"value": {"text": "sdfadf", "startDate": "2025-02-01T00:00:00.000Z", "endDate": "2025-02-28T00:00:00.000Z"}}
```

### Data
- start: number(Float type)
- end: number(Float type)
- value: 
```json
{"value": {"url": "https://example.com", "text": "sdafsdf"}}
```

### Category
- start: number(Float type)
- end: number(Float type)
- value:
```json
{"value": {"color": "#F1A73D", "slug": "memory", "value": "Memory and the Imaginary"}}
```
```js
// Category data list
[
   {
       "slug": "identity",
       "color": "#9E21E8",
   },
   {
       "slug": "knowledge",
       "color": "#8BA5F8",
   },
   {
       "slug": "artistic_reflection",
       "color": "#691220",
   },
   {
       "slug": "restitution",
       "color": "#EC6735",
   },
   {
       "slug": "memory",
       "color": "#F1A73D",
   },
]
```