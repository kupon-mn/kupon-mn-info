# 📝 Problem Statement and Project Introduction: Discount Discovery Platform

## 1. Introduction (Context)

Modern consumers actively seek discounts and promotions for everyday goods and services. However, this information is often scattered across multiple channels such as social media, news websites, and individual business websites. This fragmentation creates difficulties for users, leading to wasted time and obstacles in finding the best deals available to them. Conversely, businesses lack an automated and effective solution for announcing short-term, targeted discounts, and for monitoring and authenticating their execution (coupon redemption).

---

## 2. The Issue (The Problem)

- **Information Fragmentation:** Consumers lack a centralized, easy-to-use platform where discount information is aggregated and can be viewed with a smooth, engaging scrolling experience (similar to Instagram).
- **Discount Authentication Process:** Businesses lack a robust mechanism for issuing and limiting coupons, and for quickly and reliably authenticating coupons at the point of service (e.g., using QR codes).
- **Search and Discovery:** Consumers are limited to traditional category and keyword searches. There is a demand for a smarter search system that allows users to express their requests using natural language and immediately find relevant discounts.

---

## 3. Proposed Solution (The Solution)

We will develop a **"Discount Discovery Platform," or Deal Aggregator App**, that unites consumers and businesses to enable smart discovery of discounts and coupons. This platform will easily deliver discount information to users and provide businesses with a tool to manage their discount marketing effectively.

---

## 4. Project Scope and Core Functions

| Stakeholder                  | Core Functions                                                                                                                                                                                                                  | Additional Requirements/Notes                                                                          |
| :--------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :----------------------------------------------------------------------------------------------------- |
| **Consumer**                 | 1. **Discount Discovery Interface:** The platform must allow users to scroll (Swipe/Scroll) through discount information in a format similar to Instagram stories or posts on both mobile and web applications.                 | A user-friendly, fast-loading UI/UX is crucial.                                                        |
|                              | 2. **Coupon Acquisition:** Users must be able to acquire coupons for selected discounted products/services, with the coupon quantity potentially being limited or unlimited.                                                    | Acquired coupons must be saved to the user's profile.                                                  |
|                              | 3. **Intelligent Search:** Utilizing the Gemini API, users can submit requests in natural language, such as "I need a haircut, show me discounted barbershops."                                                                 | The system must list the top 5 most relevant results and offer a "More" button to display all results. |
| **Business**                 | 4. **Discount Entry System:** Businesses must be able to input information for their discounted products and services (name, price, discount percentage/amount, duration, description, images) and the total number of coupons. | Category information for products/services must be defined.                                            |
|                              | 5. **Coupon Redemption:** When a customer arrives to use a discount, the business must authenticate the user's coupon by scanning a QR code and marking the coupon as redeemed.                                                 | This ensures a reliable mechanism against coupon double-use.                                           |
|                              | 6. **External Linkage:** Discounted product and service information will link to the relevant business's website address.                                                                                                       | This is intended to simplify the purchase process.                                                     |
| **Technology & Performance** | 7. **Backend Architecture:** The backend will be developed using FastAPI (Python).                                                                                                                                              | Must be capable of handling a high volume of concurrent requests quickly and reliably.                 |
|                              | 8. **AI Integration:** The intelligent search function will be implemented using the Google Gemini API.                                                                                                                         |                                                                                                        |

---

## 5. Technology Choices (Technology Stack)

| Component                 | Selected Technology      | Rationale                                                                                                    |
| :------------------------ | :----------------------- | :----------------------------------------------------------------------------------------------------------- |
| **Backend Framework**     | FastAPI (Python)         | Supports asynchronous operations, offers high performance, and leverages the Python ecosystem.               |
| **Intelligent Search/AI** | Google Gemini API        | To understand natural language input and suggest the most appropriate discounts based on the user's request. |
| **Frontend**              | (To be Determined)       | Responsive technology suitable for web and mobile (e.g., React, Next.js, or Vue.js).                         |
| **Database**              | (Additional Requirement) | To store discount, coupon, user, and business data.                                                          |

---

## 6. Additional Recommendations

For the successful development of the project, it is essential to prioritize the security of user and business data, as well as the mechanism for monitoring and preventing duplicate coupon usage. To fully utilize FastAPI's asynchronous capabilities, the part of the application that interacts with the database (ORM) should also be organized asynchronously.
