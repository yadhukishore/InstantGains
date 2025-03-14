## Previous Code in simple mongoDB Methods
```typescript
export const getAllCourses = async (req: Request, res: Response) => {
  try {
    const { page = 1, limit = 10 } = req.query;
    const skip = (parseInt(page as string) - 1) * parseInt(limit as string);

    const courses = await Course.find({ isDelete: false })
      .select('name description thumbnail price level category estimatedPrice')
      .skip(skip)
      .limit(parseInt(limit as string));

    const totalCourses = await Course.countDocuments({ isDelete: false });

    res.status(200).json({
      success: true,
      courses,
      totalPages: Math.ceil(totalCourses / parseInt(limit as string)),
      currentPage: parseInt(page as string),
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: (error as Error).message,
    });
  }
}
```

## Updated code with aggregation Pipeline

```javascript
import { Request, Response } from 'express';
import Course from '../models/Course'; // Adjust the import according to your file structure

export const getAllCourses = async (req: Request, res: Response) => {
  try {
    const { page = 1, limit = 10 } = req.query;
    const skip = (parseInt(page as string) - 1) * parseInt(limit as string);

    const coursesAggregation = [
      {
        $match: { isDelete: false } // Match documents that are not deleted
      },
      {
        $project: {
          name: 1,
          description: 1,
          thumbnail: 1,
          price: 1,
          level: 1,
          category: 1,
          estimatedPrice: 1
        }
      },
      {
        $skip: skip // Skip the documents for pagination
      },
      {
        $limit: parseInt(limit as string) // Limit the number of documents per page
      }
    ];

    const [courses, totalCourses] = await Promise.all([
      Course.aggregate(coursesAggregation),
      Course.countDocuments({ isDelete: false })
    ]);

    res.status(200).json({
      success: true,
      courses,
      totalPages: Math.ceil(totalCourses / parseInt(limit as string)),
      currentPage: parseInt(page as string),
    });
  } catch (error) {
    res.status(500).json({
      success: false,
      message: (error as Error).message,
    });
  }
};


```
### TS;
```typescript
export const getAllCourses = async (req: Request, res: Response): Promise<void> => {
  try {
    // Parse page and limit from query parameters, default to 1 and 10 respectively
    const page = parseInt(req.query.page as string, 10) || 1;
    const limit = parseInt(req.query.limit as string, 10) || 10;
    const skip = (page - 1) * limit;

    // Define the aggregation pipeline
    const coursesAggregation = [
      {
        $match: { isDelete: false } // Filter out deleted courses
      },
      {
        $project: {
          name: 1,
          description: 1,
          thumbnail: 1,
          price: 1,
          level: 1,
          category: 1,
          estimatedPrice: 1
        }
      },
      {
        $skip: skip // Skip the appropriate number of documents
      },
      {
        $limit: limit // Limit the number of documents to return
      }
    ];

    // Execute aggregation and count documents
    const [courses, totalCourses] = await Promise.all([
      Course.aggregate(coursesAggregation).exec(),
      Course.countDocuments({ isDelete: false }).exec()
    ]);

    // Send response with paginated courses and pagination metadata
    res.status(200).json({
      success: true,
      courses,
      totalPages: Math.ceil(totalCourses / limit),
      currentPage: page
    });
  } catch (error) {
    // Handle any errors
    res.status(500).json({
      success: false,
      message: (error as Error).message
    });
  }
};
```



### Frontend.tsx

```tsx

// src/components/user/UserCourseList.tsx
import React, { useCallback, useEffect, useState } from 'react';
import { Link } from 'react-router-dom';
import axios from 'axios';
import { motion } from 'framer-motion';
import Header from './HeaderUser';
import CurrentLearningCourses from './CurrentLearningCourses';
import { useSelector, useDispatch } from 'react-redux';
import { RootState } from '../../store/store';
import { addToWishlist, removeFromWishlist, setWishlist } from '../../../features/wishlist/wishlistSlice';
import { FaHeart, FaRegHeart } from 'react-icons/fa';
import { Pagination } from 'flowbite-react';

interface Course {
  _id: string;
  name: string;
  description: string;
  thumbnail: {
    url: string;
  };
  price: number;
  estimatedPrice: number;
  level: string;
  category: string;
}

interface CurrentCourse extends Course {
  progress: number;
}

const UserCourseList: React.FC = () => {
  const user = useSelector((state: RootState) => state.auth.user);
  const wishlist = useSelector((state: RootState) => state.wishlist.items);
  const dispatch = useDispatch();

  const [courses, setCourses] = useState<Course[]>([]);
  const [currentCourses, setCurrentCourses] = useState<CurrentCourse[]>([]);
  const [loading, setLoading] = useState(true);
  const [currentPage, setCurrentPage] = useState(1);
  const [totalPages, setTotalPages] = useState(1);
  const coursesPerPage = 6; 

  useEffect(() => {
    const fetchCourses = async () => {
      try {
        const [allCoursesResponse, currentCoursesResponse] = await Promise.all([
          axios.get(`http://localhost:8000/api/user/courses?page=${currentPage}&limit=${coursesPerPage}`),
          axios.get(`http://localhost:8000/api/user/current-courses?userId=${user?.id}`)
        ]);

        setCourses(allCoursesResponse.data.courses);
        setTotalPages(allCoursesResponse.data.totalPages);
        setCurrentCourses(currentCoursesResponse.data.currentCourses);

        if (user?.id) {
          const wishlistResponse = await axios.get(`http://localhost:8000/api/user/wishlist/${user.id}`);
          dispatch(setWishlist(wishlistResponse.data.wishlist));
        }

        setLoading(false);
      } catch (error) {
        console.error('Error fetching courses or wishlist:', error);
        setLoading(false);
      }
    };

    if (user?.id) {
      fetchCourses();
    }
  }, [user, dispatch, currentPage]);

  const handleWishlistToggle = useCallback(async (course: Course) => {
    const isWishlisted = wishlist.some(item => item._id === course._id);

    try {
      const url = isWishlisted ? '/wishlist/remove' : '/wishlist/add';
      await axios.post(`http://localhost:8000/api/user${url}`, {
        userId: user?.id,
        courseId: course._id,
      });
      dispatch(isWishlisted ? removeFromWishlist(course._id) : addToWishlist(course));
    } catch (error) {
      console.error('Error updating wishlist:', error);
    }
  }, [wishlist, user, dispatch]);

  const onPageChange = (page: number) => {
    setCurrentPage(page); 
  };

  if (loading) {
    return (
      <div className="min-h-screen bg-[#071A2B] text-white flex items-center justify-center">
        <div className="animate-spin rounded-full h-32 w-32 border-t-2 border-b-2 border-blue-500"></div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-[#071A2B] text-white">
      <Header />
      <main className="max-w-7xl mx-auto py-12 px-4 sm:px-6 lg:px-8">
        <CurrentLearningCourses courses={currentCourses} />
        <h1 className="text-4xl font-bold mb-8 text-center">
          <span className="bg-clip-text text-transparent bg-gradient-to-r from-pink-500 to-yellow-500">
            Explore Our Courses
          </span>
        </h1>
        <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-8">
          {courses.map((course, index) => {
            const discount = course.estimatedPrice
              ? Math.round(((course.estimatedPrice - course.price) / course.estimatedPrice) * 100)
              : 0;

            const isWishlisted = wishlist.some(item => item._id === course._id);

            return (
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
                    {discount > 0 && (
                      <span className="text-sm font-semibold text-red-400 ml-4">{discount}% Off</span>
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
                      onClick={() => handleWishlistToggle(course)}
                      className="ml-4 text-pink-500 hover:text-pink-600 transition duration-300"
                      aria-label="Toggle Wishlist"
                    >
                      {isWishlisted ? <FaHeart size={24} /> : <FaRegHeart size={24} />}
                    </button>
                  </div>
                </div>
              </motion.div>
            );
          })}
        </div>

        {/* Pagination Component */}
        <div className="flex justify-center mt-8">
          <Pagination
            currentPage={currentPage}
            totalPages={totalPages}
            onPageChange={onPageChange}
          />
        </div>
      </main>
    </div>
  );
};

export default UserCourseList;

```
