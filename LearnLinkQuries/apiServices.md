## Created a API Services for making the API call in a better way.


```typescript
// src/services/api.ts
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios';

const BASE_URL = 'http://localhost:8000/api';

class ApiService {
  private api: AxiosInstance;

  constructor() {
    this.api = axios.create({
      baseURL: BASE_URL,
      headers: {
        'Content-Type': 'application/json',
      },
    });

    this.setupInterceptors();
  }

  private setupInterceptors(): void {
    this.api.interceptors.request.use(
      (config) => {
        const token = localStorage.getItem('accessToken');
        if (token) {
          config.headers['Authorization'] = `Bearer ${token}`;
        }
        return config;
      },
      (error) => Promise.reject(error)
    );

    this.api.interceptors.response.use(
      (response) => response,
      async (error) => {
        const originalRequest = error.config;
        if (error.response.status === 401 && !originalRequest._retry) {
          originalRequest._retry = true;
          try {
            const refreshToken = localStorage.getItem('refreshToken');
            const response = await axios.post(`${BASE_URL}/refresh-token`, { refreshToken });
            const { accessToken } = response.data;
            localStorage.setItem('accessToken', accessToken);
            this.api.defaults.headers['Authorization'] = `Bearer ${accessToken}`;
            return this.api(originalRequest);
          } catch (refreshError) {
            localStorage.removeItem('accessToken');
            localStorage.removeItem('refreshToken');
            // Redirect to login or show error
            return Promise.reject(refreshError);
          }
        }
        return Promise.reject(error);
      }
    );
  }

  public async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response: AxiosResponse<T> = await this.api.get(url, config);
    return response.data;
  }

  public async post<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response: AxiosResponse<T> = await this.api.post(url, data, config);
    return response.data;
  }

  public async put<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    const response: AxiosResponse<T> = await this.api.put(url, data, config);
    return response.data;
  }

  public async delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
    const response: AxiosResponse<T> = await this.api.delete(url, config);
    return response.data;
  }

  public createCancelToken() {
    return axios.CancelToken.source();
  }
}

export const apiService = new ApiService();

```
### Explain:-

The `api.ts` file you've created is a service that abstracts and manages API requests in your application using Axios, a popular JavaScript library for making HTTP requests. Let's break down the code step by step, explain its purpose, uses, advantages, and where else you can use such a service.

### Overview of the Code

1. **Imports and Constants:**
   ```typescript
   import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from 'axios';

   const BASE_URL = 'http://localhost:8000/api';
   ```
   - **Purpose:** Import Axios and relevant types for better type checking in TypeScript.
   - **Usage:** Define a `BASE_URL` that will be used as the base path for all API requests.
   - **Advantage:** Centralizes the API endpoint, making it easy to update if the API base URL changes.
   - **Other Uses:** You can set different base URLs for development, testing, and production environments using environment variables.

2. **Class Definition:**
   ```typescript
   class ApiService {
     private api: AxiosInstance;

     constructor() {
       this.api = axios.create({
         baseURL: BASE_URL,
         headers: {
           'Content-Type': 'application/json',
         },
       });

       this.setupInterceptors();
     }
   }
   ```
   - **Purpose:** Define a class `ApiService` with a private `AxiosInstance` property called `api`.
   - **Usage:** The `constructor` initializes the Axios instance with a base URL and default headers (`Content-Type: application/json`).
   - **Advantage:** Encapsulates Axios setup and provides a consistent configuration for all API calls.
   - **Other Uses:** This pattern is useful for services that need consistent API setups, such as authentication or data fetching services.

3. **Interceptors Setup:**
   ```typescript
   private setupInterceptors(): void {
     this.api.interceptors.request.use(
       (config) => {
         const token = localStorage.getItem('accessToken');
         if (token) {
           config.headers['Authorization'] = `Bearer ${token}`;
         }
         return config;
       },
       (error) => Promise.reject(error)
     );

     this.api.interceptors.response.use(
       (response) => response,
       async (error) => {
         const originalRequest = error.config;
         if (error.response.status === 401 && !originalRequest._retry) {
           originalRequest._retry = true;
           try {
             const refreshToken = localStorage.getItem('refreshToken');
             const response = await axios.post(`${BASE_URL}/refresh-token`, { refreshToken });
             const { accessToken } = response.data;
             localStorage.setItem('accessToken', accessToken);
             this.api.defaults.headers['Authorization'] = `Bearer ${accessToken}`;
             return this.api(originalRequest);
           } catch (refreshError) {
             localStorage.removeItem('accessToken');
             localStorage.removeItem('refreshToken');
             // Redirect to login or show error
             return Promise.reject(refreshError);
           }
         }
         return Promise.reject(error);
       }
     );
   }
   ```
   - **Purpose:** To set up request and response interceptors for handling authorization and error management.
   - **Request Interceptor:** Attaches the `Authorization` header with the access token for each outgoing request if the token exists.
   - **Response Interceptor:** Handles 401 errors (unauthorized). It attempts to refresh the token using a refresh token and retries the original request.
   - **Advantages:**
     - **Centralized Error Handling:** Simplifies error management by handling common errors in one place.
     - **Token Management:** Automatically refreshes access tokens without user intervention, enhancing the user experience.
   - **Other Uses:** Interceptors are useful in applications where you need to:
     - Retry requests on failure.
     - Log requests/responses.
     - Manage global loading states.

4. **CRUD Methods:**
   ```typescript
   public async get<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
     const response: AxiosResponse<T> = await this.api.get(url, config);
     return response.data;
   }

   public async post<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
     const response: AxiosResponse<T> = await this.api.post(url, data, config);
     return response.data;
   }

   public async put<T>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
     const response: AxiosResponse<T> = await this.api.put(url, data, config);
     return response.data;
   }

   public async delete<T>(url: string, config?: AxiosRequestConfig): Promise<T> {
     const response: AxiosResponse<T> = await this.api.delete(url, config);
     return response.data;
   }
   ```
   - **Purpose:** Define generic methods for GET, POST, PUT, and DELETE requests.
   - **Usage:** Each method sends the respective HTTP request using the configured Axios instance and returns the response data.
   - **Advantage:** Abstracts the actual HTTP calls, making it easy to switch implementations if needed (e.g., switching from Axios to Fetch).
   - **Other Uses:** You can extend these methods to include PATCH, HEAD, or other HTTP methods as required.

5. **Cancel Token:**
   ```typescript
   public createCancelToken() {
     return axios.CancelToken.source();
   }
   ```
   - **Purpose:** Provides a way to cancel ongoing requests using Axios's cancel token feature.
   - **Usage:** Use this to abort requests when components unmount or when a user navigates away from a page.
   - **Advantage:** Helps to avoid memory leaks and unnecessary API calls.
   - **Other Uses:** Useful in scenarios like search inputs where you cancel the previous request as the user types.

### Advantages of `api.ts`

- **Code Reusability:** Centralizes all API calls in one place, reducing code duplication.
- **Scalability:** Makes it easy to add new endpoints or change API configurations without touching multiple files.
- **Consistency:** Ensures all requests have a uniform structure, making debugging and enhancements easier.
- **Maintainability:** Changes in the API (like endpoint changes or headers) can be handled in one location.

### Where Else Can You Use This?

- **In Other Services:** This pattern can be replicated for different microservices or modules (e.g., UserService, ProductService).
- **In Different Projects:** Any project that involves API calls can benefit from this structured approach.
- **With Other HTTP Libraries:** You can adapt this pattern for Fetch API, SuperAgent, or other libraries with minimal changes.

Overall, the `api.ts` file you created encapsulates all aspects of API interaction, offering a clean and maintainable approach to managing server communication in your application.
