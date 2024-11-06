 - The authorizeUser middleware is responsible for verifying the user's authentication token.
 - It checks if the authorization header is present in the request.
 - It extracts the token from the header and verifies it using the JWT_SECRET environment variable.
 - If the token is valid, it fetches the user from the database and assigns the user's ID to the req.userId property.
 - The isLoggedIn middleware is similar to authorizeUser, but it also sends a response with the user's data if the token is valid.

```javascript

import { AppError, AppResponse } from "../utils/appUtils.js";
import User from "../models/userModel.js";
import jwt from "jsonwebtoken";

export const authorizeUser = (req, res, next) => {
  if (!req.headers.authorization) {
    throw new AppError(401, "Token do not exist");
  }

  const [_, token] = req.headers.authorization.split(" ");

  jwt.verify(token, process.env.JWT_SECRET, async (err, decoded) => {
    if (err) {
      throw new AppError(401, "Unauthorized: Invalid token");
    }

    const user = await User.findOne({ _id: decoded.userId });

    if (!user) throw new AppError(401, "Unauthorized: User not found");

    req.userId = decoded.userId;

    next();
  });
};

export const isLoggedIn = (req, res, next) => {
  if (!req.headers.authorization) {
    throw new AppError(401, "Token do not exist");
  }

  const [_, token] = req.headers.authorization.split(" ");

  jwt.verify(token, process.env.JWT_SECRET, async (err, decoded) => {
    if (err) {
      throw new AppError(401, "Unauthorized: Invalid token");
    }

    const user = await User.findOne({ _id: decoded.userId });

    if (!user) throw new AppError(401, "Unauthorized: User not found");

    req.userId = decoded.userId;

    new AppResponse(res, 200, "User is logged-in", {
      userData: {
        isUser: true,
        email: user.email,
      },
    });
  });
};
