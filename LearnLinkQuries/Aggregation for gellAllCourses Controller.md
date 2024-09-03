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
