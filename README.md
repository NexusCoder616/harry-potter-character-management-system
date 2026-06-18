# Harry Potter Characters API

A REST API to manage Harry Potter characters — built with Node.js, Express, MongoDB, and Cloudinary for image uploads.

![Node.js](https://img.shields.io/badge/Node.js-18.x-green) ![Express](https://img.shields.io/badge/Express-5.x-lightgrey) ![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose-brightgreen) ![JWT](https://img.shields.io/badge/Auth-JWT-orange) ![Cloudinary](https://img.shields.io/badge/Images-Cloudinary-blue)

---

## Getting Started

Clone the repo and install dependencies:

```bash
git clone <your-repo-url>
cd harrypotterbackend
npm install
```

Create a `.env` file in the root:

```env
PORT=3001
MONGO_URI=mongodb+srv://<user>:<password>@cluster.mongodb.net/harrypotter
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
```

Start the server:

```bash
npm run dev    # with nodemon (recommended for development)
npm start      # production
```

---

## Project Structure

```
harrypotterbackend/
├── config/
│   ├── db.js                  # MongoDB connection
│   └── cloudinary.js          # Cloudinary setup
├── controller/
│   ├── authController.js      # register & login
│   └── characterController.js # CRUD for characters
├── middleware/
│   └── authMiddleware.js      # JWT token check
├── models/
│   ├── Auth.js                # User schema
│   └── Character.js           # Character schema
├── router/
│   ├── authRouter.js
│   └── characterRoute.js
├── uploads/                   # temp files before Cloudinary upload
├── .env
└── index.js
```

---

## Authentication

Login returns a JWT token. For protected routes, add it to the `Authorization` header — no `Bearer` prefix, just the raw token:

```
Authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Tokens expire in **7 days**.

---

## API Reference

Base URL: `http://localhost:3001`

### Auth

#### `POST /auth/register`

Register a new user.

**Request body:**
```json
{
  "name": "Harry Potter",
  "email": "harry@hogwarts.com",
  "password": "expelliarmus123"
}
```

**201 response:**
```json
{
  "message": "register done......",
  "user": {
    "_id": "664abc123def456789012345",
    "name": "Harry Potter",
    "email": "harry@hogwarts.com",
    "password": "$2a$10$...",
    "__v": 0
  }
}
```

Returns `400` with `"user already registered"` if the email exists.

---

#### `POST /auth/login`

Login and get a token.

**Request body:**
```json
{
  "email": "harry@hogwarts.com",
  "password": "expelliarmus123"
}
```

**200 response:**
```json
{
  "message": "login done...",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

Returns `400` if the user isn't found or the password is wrong.

---

### Characters

#### `POST /api/v1/add/character`

Add a new character. You can either upload an image file or pass a Cloudinary URL directly.

**Option 1 — file upload** (`multipart/form-data`):

| field | type   |
|-------|--------|
| name  | string |
| house | string |
| wand  | string |
| image | file   |

The file gets uploaded to Cloudinary automatically (stored under the `harrypotter` folder).

**Option 2 — image URL** (`application/json`):

```json
{
  "name": "Hermione Granger",
  "house": "Gryffindor",
  "wand": "10¾ inch, vine wood, dragon heartstring core",
  "image": "https://res.cloudinary.com/demo/image/upload/sample.jpg"
}
```

**201 response:**
```json
{
  "success": true,
  "message": "Character Added Successfully",
  "character": {
    "_id": "664abc123def456789000001",
    "name": "Hermione Granger",
    "house": "Gryffindor",
    "wand": "10¾ inch, vine wood, dragon heartstring core",
    "image": "https://res.cloudinary.com/yourcloud/image/upload/harrypotter/abc123.jpg",
    "__v": 0
  }
}
```

Returns `400` if neither an image file nor a URL is provided.

---

#### `GET /api/v1/get/characters`

Get all characters. **Requires auth token.**

**200 response:**
```json
{
  "success": true,
  "totalCharacters": 2,
  "characters": [
    {
      "_id": "664abc123def456789000001",
      "name": "Harry Potter",
      "house": "Gryffindor",
      "wand": "11 inch, holly, phoenix feather core",
      "image": "https://res.cloudinary.com/yourcloud/image/upload/harrypotter/harry.jpg",
      "__v": 0
    },
    {
      "_id": "664abc123def456789000002",
      "name": "Draco Malfoy",
      "house": "Slytherin",
      "wand": "10 inch, hawthorn, unicorn hair core",
      "image": "https://res.cloudinary.com/yourcloud/image/upload/harrypotter/draco.jpg",
      "__v": 0
    }
  ]
}
```

Returns `400` with `"login first"` if no token is provided.

---

#### `GET /api/v1/get/characters/:id`

Get a single character by ID.

**200 response:**
```json
{
  "success": true,
  "character": {
    "_id": "664abc123def456789000003",
    "name": "Ron Weasley",
    "house": "Gryffindor",
    "wand": "14 inch, willow, unicorn hair core",
    "image": "https://res.cloudinary.com/yourcloud/image/upload/harrypotter/ron.jpg",
    "__v": 0
  }
}
```

Returns `404` if no character matches that ID.

---

#### `PUT /api/v1/update/character/:id`

Update a character's name, house, or wand. Image can't be changed through this route.

**Request body:**
```json
{
  "name": "Neville Longbottom",
  "house": "Gryffindor",
  "wand": "13 inch, cherry, unicorn hair core"
}
```

All fields are optional — only include what you want to update.

**200 response:**
```json
{
  "success": true,
  "message": "Character Updated Successfully",
  "character": {
    "_id": "664abc123def456789000005",
    "name": "Neville Longbottom",
    "house": "Gryffindor",
    "wand": "13 inch, cherry, unicorn hair core",
    "image": "https://res.cloudinary.com/yourcloud/image/upload/harrypotter/neville.jpg",
    "__v": 0
  }
}
```

Returns `404` if character not found.

---

#### `DELETE /api/v1/delete/character/:id`

Delete a character permanently.

**200 response:**
```json
{
  "success": true,
  "message": "Character Deleted Successfully"
}
```

Returns `404` if character not found.

---

## Routes at a Glance

| Method | Route | Auth? |
|--------|-------|-------|
| POST | `/auth/register` | No |
| POST | `/auth/login` | No |
| POST | `/api/v1/add/character` | No |
| GET | `/api/v1/get/characters` | **Yes** |
| GET | `/api/v1/get/characters/:id` | No |
| PUT | `/api/v1/update/character/:id` | No |
| DELETE | `/api/v1/delete/character/:id` | No |

---

## A Few Things Worth Knowing

- Passwords are hashed with bcrypt before being stored.
- The JWT secret is currently hardcoded as `"mykeypswrd"` — move it to `.env` before pushing to production.
- Local uploads in the `uploads/` folder are just a temp step before the file goes to Cloudinary. They're not served by the API.
- Right now only the GET all characters route is protected. Depending on your use case, you'll probably want to add auth to the add/update/delete routes too.

---

## Stack

- **Express 5** — routing
- **Mongoose 8** — MongoDB ODM
- **bcryptjs** — password hashing
- **jsonwebtoken** — auth tokens
- **Cloudinary** — image storage
- **Multer** — handling file uploads
- **dotenv** — environment config
- **nodemon** — dev server
