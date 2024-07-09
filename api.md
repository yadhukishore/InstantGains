You're right. Let's include the missing routes for managing the student profile and changing the password.

### API Specification

### Mod1: User Navigation

#### Frontend

**Home Page**

| Method | URL         | Description          | Response              |
|--------|-------------|----------------------|-----------------------|
| GET    | /           | Get Landing page     | Render landing page   |

**Courses**

| Method | URL                   | Description                 | Response                       |
|--------|-----------------------|-----------------------------|--------------------------------|
| GET    | /courses              | Get all courses             | Render all courses page        |
| GET    | /courses/:id          | Get course details          | Render course details page     |
| POST   | /courses/:id/cart     | Add course to cart          | Update cart                    |
| POST   | /courses/:id/wishlist | Add course to wishlist      | Update wishlist                |
| POST   | /courses/:id/purchase | Purchase course             | Process payment (Razorpay)     |
| GET    | /courses/:id/watch    | Watch purchased course      | Render course video player     |
| POST   | /courses/:id/notes    | Create personal notes       | Save notes                     |
| POST   | /courses/:id/shared-notes | Create shared notes         | Save shared notes              |
| POST   | /courses/:id/schedule-call | Schedule a live video call | Schedule call, notify teacher  |

**Books**

| Method | URL                   | Description                 | Response                       |
|--------|-----------------------|-----------------------------|--------------------------------|
| GET    | /books                | Get all books               | Render all books page          |
| GET    | /books/:id            | Get book details            | Render book details page       |
| POST   | /books/:id/cart       | Add book to cart            | Update cart                    |
| POST   | /books/:id/wishlist   | Add book to wishlist        | Update wishlist                |
| POST   | /books/:id/purchase   | Purchase book               | Process payment (Razorpay)     |

### Mod2: Authentication

**Login/Signup**

| Method | URL     | Description        | Response                  |
|--------|---------|--------------------|---------------------------|
| GET    | /login  | Render login page  | Render login form         |
| POST   | /login  | Authenticate user  | Redirect to feed page     |
| GET    | /signup | Render signup page | Render signup form        |
| POST   | /signup | Register new user  | Redirect to feed page     |
| POST   | /logout | Logout user        | Redirect to landing page  |

### Mod3: User Feed

**Feed**

| Method | URL                 | Description            | Response                       |
|--------|---------------------|------------------------|--------------------------------|
| GET    | /feed               | Get user feed          | Render user feed with posts    |
| POST   | /feed               | Create new post        | Update feed                    |
| GET    | /feed/:postId       | Get post details       | Render post details            |
| POST   | /feed/:postId/comment | Add comment to post    | Update comments                |

### Mod4: User Profile

**Profile**

| Method | URL           | Description            | Response                       |
|--------|---------------|------------------------|--------------------------------|
| GET    | /profile      | Get user profile       | Render profile page            |
| PUT    | /profile/edit | Edit user profile      | Update profile                 |
| PUT    | /profile/change-password | Change password        | Update password                |

### Mod5: Notes

**Notes**

| Method | URL           | Description            | Response                       |
|--------|---------------|------------------------|--------------------------------|
| GET    | /notes        | Get all personal notes | Render personal notes page     |
| GET    | /notes/:id    | Get personal note      | Render note details page       |
| POST   | /notes        | Create new note        | Save new note                  |
| PUT    | /notes/:id    | Update personal note   | Update note                    |
| DELETE | /notes/:id    | Delete personal note   | Remove note                    |

### Mod6: Shared Notes

**Shared Notes**

| Method | URL                   | Description                 | Response                       |
|--------|-----------------------|-----------------------------|--------------------------------|
| GET    | /shared-notes         | Get all shared notes        | Render shared notes page       |
| GET    | /shared-notes/:id     | Get shared note details     | Render shared note details     |
| POST   | /shared-notes         | Create new shared note      | Save new shared note           |
| PUT    | /shared-notes/:id     | Update shared note          | Update shared note             |
| DELETE | /shared-notes/:id     | Delete shared note          | Remove shared note             |

### Mod7: Video Call

**Video Call**

| Method | URL                   | Description                 | Response                       |
|--------|-----------------------|-----------------------------|--------------------------------|
| POST   | /video-call/schedule  | Schedule a video call       | Notify teacher, set call time  |
| GET    | /video-call/join      | Join scheduled video call   | Start video call               |
| POST   | /video-call/accept    | Accept scheduled video call | Confirm call time              |

### Response Formats

- **Success Response:**
  ```json
  {
    "status": "success",
    "data": { ... }
  }
  ```

- **Error Response:**
  ```json
  {
    "status": "error",
    "message": "Error message"
  }
  ```

This updated API documentation now includes routes for managing student profiles and changing passwords.
