# Event Ticketing System

## Project Overview

The Event Ticketing System is a web-based application designed to simplify the process of browsing, purchasing, and managing event tickets. Users can easily view available events, purchase tickets by providing their contact details, and retrieve their previously purchased tickets using their contact information. The system ensures fair access to tickets by enforcing purchase limits per user and generates unique QR codes for each ticket to facilitate secure entry at events.

## Technologies Used

- **Frontend:** Next.js
- **Backend:** Node.js with Express.js
- **Database:** PostgreSQL with Drizzle ORM
- **Styling:** DaisyUI
- **State Management:** Zustand
- **Data Fetching and Caching:** TanStack Query

## Steps to Run the Project Locally

### Backend Setup

1.  **Install Dependencies:**

    ```bash
    pnpm install
    ```

2.  **Environment Variables:** Create a `.env` file in the backend directory and populate it with the following environment variables:

    ```
    PORT=<port_number>
    DATABASE_URL=<neon_postgres_connection_string>  # Use Neon Postgres for the database
    JWT_SECRET=<your_jwt_secret>
    FROM_EMAIL=<your_email_address>
    SENDGRID_KEY=<your_sendgrid_api_key>
    TWILLO_TOKEN=<your_twilio_auth_token>
    TWILLO_SID=<your_twilio_account_sid>
    TWILLO_FROM=<your_twilio_phone_number>
    RAZORPAY_KEY=<your_razorpay_key>
    RAZORPAY_SECRET=<your_razorpay_secret>
    ```

    Remember to use a Neon Postgres connection string for `DATABASE_URL`.

3.  **Database Migrations:** Apply database migrations using Drizzle:

    ```bash
    npx drizzle-kit push
    ```

4.  **Run the Backend:**
    ```bash
    pnpm run dev
    ```

### Frontend Setup

1.  **Install Dependencies:**

    ```bash
    npm i
    ```

2.  **Environment Variables:** Create a `.env` file in the frontend directory and add the backend URL:

    ```
    NEXT_PUBLIC_BACKEND_URL=<backend_url>  # e.g., http://localhost:<port_number>
    NEXT_PUBLIC_RAZORPAY_KEY=<your_razorpay_key>
    ```

3.  **Run the Frontend:**
    ```bash
    npm run dev
    ```

## API Endpoints (All endpoints begin with `/api/v1/`)

### Admin Endpoints

- **`POST /admin/register`**: Registers a new admin user. Requires a username and password in the request body. Returns a 201 status code with a success message upon successful registration. Handles validation errors and password hashing before creating the admin user.
- **`POST /admin/login`**: Authenticates an admin user. Requires a username and password in the request body. If authentication is successful, it returns a JWT token in the response. Handles invalid username/password scenarios.

- **`GET /admin/tickets-sold/:id`**: Retrieves the number of tickets sold for a specific event. Requires admin login (JWT token in the Authorization header). The event ID is passed as a path parameter. Returns a JSON response containing the count of tickets sold. Validates the event ID.

- **`GET /admin/remaining-tickets/:id`**: Retrieves the number of remaining tickets for a specific event. Requires admin login (JWT token in the Authorization header). The event ID is passed as a path parameter. Returns a JSON response containing the count of remaining tickets. Validates the event ID.

- **`PATCH /cancel-event/:id`**: Cancels a specific event. Requires admin login (JWT token in the Authorization header). The event ID is passed as a path parameter. Updates the event's `isActive` status to `false`. Returns a success message upon successful cancellation. Validates the event ID.

### User Endpoints

- **`POST users/check-contact`**: Checks if a user with the provided email or phone number exists and is verified. If the user exists and is verified, it returns a 200 status code with a "Contact verified" message and user data. If the user doesn't exist or isn't verified, it generates an OTP, sends it via email or SMS, and creates a new user record (if one doesn't exist). It then returns a 200 status code with an "OTP sent" message and the new/existing user data. Handles input validation for email/phone.

- **`POST /verify-otp`**: Verifies the OTP entered by the user. Requires the OTP in the request body. If the OTP is valid and not expired, it marks the user as verified and returns a 200 status code with a "OTP verified" message. Handles invalid or expired OTP scenarios.

### Events Endpoints

- **`POST /events`**: Creates a new event. Requires admin login (JWT token in the Authorization header). The request body must contain event details like name, description, location, date (must be a future date), total tickets, maximum tickets per user (minimum 1), and price. It validates the input data using a schema and stores the event in the database, associating it with the admin who created it. Returns a 201 status code with a success message upon successful event creation. Handles validation errors.

- **`GET /events`**: Retrieves a list of events. Supports pagination using query parameters `page` (default: 1) and `limit` (default: 10). Returns a JSON response containing an array of event objects.

- **`GET /events/:id`**: Retrieves a specific event by its ID. The event ID is passed as a path parameter. Returns a JSON response containing the event object if found. Handles cases where the event is not found. Validates the provided ID.

### Ticket Endpoints

- **`POST /tickets/validate`**: Validates a ticket purchase request. Requires `eventId`, `userId`, and `quantity` in the request body. Checks if the user is eligible to purchase the specified quantity of tickets for the given event. Returns a success message if validation passes.
- **`POST /tickets/buy`**: Purchases tickets for an event. Requires `eventId`, `userId`, `quantity`, and `paymentId` in the request body. Creates the specified number of tickets, generates QR codes for each ticket, and returns the ticket details along with the QR codes.
- **`GET /tickets`**: Retrieves tickets for a user. Requires either `email` or `phone` as a query parameter to identify the user. Returns a list of events with the user's tickets for each event, including QR codes for each ticket. Handles cases where the user is not found or not verified. Returns an error if neither email nor phone is provided.

### Payment Endpoints

- **`POST /initiateRazorpay`**: Initiates a Razorpay payment. Requires `amount`, `userId`, and `eId` (event ID) in the request body. Creates a Razorpay order and stores payment information with a "pending" status. Returns the Razorpay order details and the payment information. Handles missing user or event ID errors.
- **`POST /validate`**: Validates a Razorpay payment. Requires `razorpay_payment_id`, `razorpay_order_id`, `razorpay_signature`, and `paymentId` in the request body. Verifies the payment signature using the Razorpay secret. Updates the payment status to "success" or "failed" based on the signature verification. Returns a success message if payment is successful, throws an error if the signature is invalid.
