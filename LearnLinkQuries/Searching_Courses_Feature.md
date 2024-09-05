### SearchButton.tsx

```typescript
import React, { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import { FaSearch } from 'react-icons/fa';

const SearchButton: React.FC = () => {
  const [tooltipVisible, setTooltipVisible] = useState(false);
  const [tooltipOpacity, setTooltipOpacity] = useState(0);
  const navigate = useNavigate();

  useEffect(() => {
    const showTooltip = () => {
      setTooltipVisible(true);
      setTimeout(() => setTooltipOpacity(1), 50); // Start fade in
      setTimeout(() => {
        setTooltipOpacity(0); // Start fade out
        setTimeout(() => setTooltipVisible(false), 300); // Hide after fade out
      }, 1000);
    };

    const intervalId = setInterval(showTooltip, 2000);
    return () => clearInterval(intervalId);
  }, []);

  const handleClick = () => {
    navigate('/searchCourse');
  };

  return (
    <div className="relative inline-flex items-center">
      <button
        onClick={handleClick}
        className="bg-blue-500 hover:bg-blue-600 text-white p-2 rounded-full shadow-lg transition duration-300 transform hover:scale-110"
        aria-label="Search Courses"
      >
        <FaSearch size={24} />
      </button>
      {tooltipVisible && (
        <div 
          className="absolute left-full ml-2 px-3 py-1 bg-gray-800 text-white text-sm rounded-lg shadow-lg transition-opacity duration-300 whitespace-nowrap"
          style={{ opacity: tooltipOpacity }}
        >
          Find more courses
        </div>
      )}
    </div>
  );
};

export default SearchButton;

```

Here are the key changes and explanations:

1. Added a new state variable for tooltip opacity:
   ```typescript
   const [tooltipOpacity, setTooltipOpacity] = useState(0);
   ```
   This allows us to control the fade effect independently of the tooltip's visibility.

2. Updated the useEffect hook to handle the fading:
   ```typescript
   useEffect(() => {
     const showTooltip = () => {
       setTooltipVisible(true);
       setTimeout(() => setTooltipOpacity(1), 50); // Start fade in
       setTimeout(() => {
         setTooltipOpacity(0); // Start fade out
         setTimeout(() => setTooltipVisible(false), 300); // Hide after fade out
       }, 1000);
     };

     const intervalId = setInterval(showTooltip, 2000);
     return () => clearInterval(intervalId);
   }, []);
   ```
   This creates a smooth fade-in and fade-out effect while maintaining the original timing (showing every 2 seconds, visible for 1 second).

3. Modified the tooltip div to use the opacity state:
   ```tsx
   {tooltipVisible && (
     <div 
       className="absolute left-full ml-2 px-3 py-1 bg-gray-800 text-white text-sm rounded-lg shadow-lg transition-opacity duration-300 whitespace-nowrap"
       style={{ opacity: tooltipOpacity }}
     >
       Find more courses
     </div>
   )}
   ```
   The `transition-opacity duration-300` class ensures a smooth transition, and the `style={{ opacity: tooltipOpacity }}` directly controls the opacity based on our state.

This implementation provides a slow, fading effect for the tooltip while maintaining the original timing you specified. The tooltip will fade in quickly, stay visible for about 1 second, and then fade out slowly. The search button functionality remains unchanged.

#


To implement a search bar that lists courses as cards, similar to the `UserCourseList.tsx` page, we'll need to create both the front-end component (`SearchCourses.tsx`) and the back-end controllers for fetching and searching courses by name and category. Here's a step-by-step guide to get this done:

### 1. Backend Controllers

First, let's create the necessary backend code to handle searching courses based on the name and category. We'll add a new controller function `searchCourses` in `userCourseController.ts`:

#### `userCourseController.ts`
```typescript
// src/controllers/user/userCourseController.ts
import { Request, Response } from 'express';
import Course from '../../models/CourseModel';

export const searchCourses = async (req: Request, res: Response) => {
  try {
    const { query, page = 1, limit = 10 } = req.query;

    // Define pagination variables
    const pageNumber = parseInt(page as string, 10);
    const pageSize = parseInt(limit as string, 10);
    const skip = (pageNumber - 1) * pageSize;

    // Search for courses by name or category
    const searchCriteria = {
      $or: [
        { name: { $regex: query, $options: 'i' } },
        { category: { $regex: query, $options: 'i' } }
      ],
      isDelete: false
    };

    const courses = await Course.find(searchCriteria)
      .skip(skip)
      .limit(pageSize)
      .populate('category', 'name') // Populate category name if needed
      .exec();

    const totalCourses = await Course.countDocuments(searchCriteria);

    res.status(200).json({
      courses,
      totalPages: Math.ceil(totalCourses / pageSize),
    });
  } catch (error) {
    res.status(500).json({ error: 'An error occurred while searching for courses.' });
  }
};
```

#### Update Routes

Now, add a route for this new controller in `userCourseRoutes.ts`:

```typescript
// src/routes/userCourseRoutes.ts

// Import the new controller
import { searchCourses } from '../controllers/user/userCourseController';

// Add the search route
router.get('/courses/search', searchCourses);
```

### 2. Frontend Component: `SearchCourses.tsx`

Next, let's create the `SearchCourses.tsx` component that includes a search bar and displays the courses as cards similar to `UserCourseList.tsx`.

#### `SearchCourses.tsx`
```tsx
// src/components/user/SearchCourses.tsx
import React, { useState, useEffect } from 'react';
import { apiService } from '../../../services/api';
import { Pagination } from 'flowbite-react';
import { Link } from 'react-router-dom';
import { motion } from 'framer-motion';
import { FaHeart, FaRegHeart } from 'react-icons/fa';

interface Course {
  _id: string;
  name: string;
  description: string;
  thumbnail: {
    url: string;
  };
  price: number;
  estimatedPrice?: number;
  level: string;
  category: string;
}

interface CoursesResponse {
  courses: Course[];
  totalPages: number;
}

const SearchCourses: React.FC = () => {
  const [searchQuery, setSearchQuery] = useState('');
  const [courses, setCourses] = useState<Course[]>([]);
  const [currentPage, setCurrentPage] = useState(1);
  const [totalPages, setTotalPages] = useState(1);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    if (searchQuery) {
      fetchCourses();
    }
  }, [searchQuery, currentPage]);

  const fetchCourses = async () => {
    setLoading(true);
    try {
      const response = await apiService.get<CoursesResponse>('/user/courses/search', {
        params: { query: searchQuery, page: currentPage, limit: 6 },
      });
      setCourses(response.courses);
      setTotalPages(response.totalPages);
    } catch (error) {
      console.error('Error fetching courses:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleSearchChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    setSearchQuery(event.target.value);
    setCurrentPage(1); // Reset to the first page on new search
  };

  const onPageChange = (page: number) => {
    setCurrentPage(page);
  };

  return (
    <div className="min-h-screen bg-[#071A2B] text-white">
      <div className="max-w-7xl mx-auto py-12 px-4 sm:px-6 lg:px-8">
        <input
          type="text"
          value={searchQuery}
          onChange={handleSearchChange}
          placeholder="Search courses by name or category"
          className="w-full p-3 rounded-md text-black mb-8"
        />
        {loading ? (
          <div className="flex justify-center">
            <div className="animate-spin rounded-full h-32 w-32 border-t-2 border-b-2 border-blue-500"></div>
          </div>
        ) : (
          <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-8">
            {courses.map((course, index) => (
              <motion.div
                key={course._id}
                className="bg-gradient-to-br from-blue-600 to-purple-600 p-1 rounded-lg shadow-lg hover:shadow-2xl transition-all duration-300 transform hover:-translate-y-1"
                initial={{ opacity: 0, y: 20 }}
                animate={{ opacity: 1, y: 0 }}
                transition={{ duration: 0.5, delay: index * 0.1 }}
              >
                <div className="bg-gray-900 p-4 rounded-lg h-full flex flex-col">
                  <img src={course.thumbnail.url} alt={course.name} className="w-full h-48 object-cover rounded-md mb-4" />
                  <h2 className="text-xl font-semibold mb-2">{course.name}</h2>
                  <p className="text-sm text-gray-300 mb-4 flex-grow">{course.category}</p>
                  <div className="flex justify-start items-center mb-4">
                    <span className="text-lg font-bold text-yellow-400">₹{course.price.toFixed(2)}</span>
                    {course.estimatedPrice && (
                      <span className="text-sm text-gray-500 line-through ml-2">₹{course.estimatedPrice.toFixed(2)}</span>
                    )}
                    {course.estimatedPrice && (
                      <span className="text-sm font-semibold text-red-400 ml-4">
                        {Math.round(((course.estimatedPrice - course.price) / course.estimatedPrice) * 100)}% Off
                      </span>
                    )}
                  </div>
                  <div className="flex items-center justify-between mt-auto">
                    <Link 
                      to={`/courses/${course._id}`} 
                      className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 transition duration-300"
                    >
                      View Details
                    </Link>
                    <button
                      className="ml-4 text-pink-500 hover:text-pink-600 transition duration-300"
                      aria-label="Toggle Wishlist"
                    >
                      <FaRegHeart size={24} />
                    </button>
                  </div>
                </div>
              </motion.div>
            ))}
          </div>
        )}

        {/* Pagination Component */}
        <div className="flex justify-center mt-8">
          <Pagination
            currentPage={currentPage}
            totalPages={totalPages}
            onPageChange={onPageChange}
          />
        </div>
      </div>
    </div>
  );
};

export default SearchCourses;
```

### Key Points in `SearchCourses.tsx`:
- A search input field allows users to type in their queries, triggering the `fetchCourses` function when the query or page number changes.
- The courses are displayed in a grid layout as cards, using a similar styling approach as in `UserCourseList.tsx`.
- Pagination is implemented to navigate through the list of courses.

With this setup, you'll have a functional search feature on the front end, paired with the backend logic to search and paginate courses effectively!
