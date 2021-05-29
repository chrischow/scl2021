# Challenge 1: Multi-Channel Contacts
See the competition on [Kaggle](https://www.kaggle.com/c/scl-2021-da/leaderboard).

## Background Information
Customer service is an important element of the Shopee business, as providing a good service for our customers end-to-end is critical for business growth and brand image. Our goal is to resolve the customer’s issue within the least amount of time while requiring the least amount of customer effort.

One measure for customer effort is the number of times a customer has to approach customer service over a particular issue, this is also known as the metric “Repeat Contact Rate” or RCR. Shopee is interested in studying the RCR in order to improve the effectiveness of our customer
service.

Customers can contact customer service via various channels such as the livechat function, filling up certain forms or calling in for help. Each time a customer contacts us with a new contact method, a new ticket is automatically generated. A complication arises when the same customer contacts us using different phone numbers or email addresses resulting in multiple tickets for the same issue. Hence, our challenge here is to identify how to merge relevant tickets together to create a complete picture of the customer issue and ultimately determine the RCR.

### Task
For each ticket, identify all contacts from each user if they have the same contact information.

For the purpose of this question, assume that all contacts from the same Phone Number / Email are the same user.

### Basic Concepts
- Each Order ID represents a transaction in Shopee.
- Each Id represents the Ticket Id made to Shopee Customer Service.
- All Phone Numbers are stored without the country code and the country code can be ignored.
- Contacts represent the number of times a user reached out to us in that particular ticket (Email, Call, Livechat etc.)
- If a value is NA means that the system or agent has no record of that value.

### Examples

| Ticket | ID  | Email | Phone | Order ID | Contacts |
| :----- | :-: | :---- | :---- | :------- | :------- |
| A      | 0   | tom@gmail.com | NA | 1234 | 5 |
| B      | 1   | NA | 682212345 | 1234 | 2 |
| C      | 2   | jerry@gmail.com | 682212345 | NA | 4 |
| D      | 3   | jerry@gmail.com | NA | NA | 3 |

Each of these tickets are related either directly or indirectly through Email, Phone or Order ID, therefore each ticket belongs to the same user.

- Ticket A and B are linked through Order ID
- Tickets B and C are linked through Phone
- Tickets C and D are linked through Email
- Tickets A and D are indirectly linked through tickets A > B > C > D

## Solution Concept

### Data Structure
We noted that the contacts data exhibited a **graphical** structure with many-to-many relations:

- Each user could have multiple emails and phones
- Each email could be associated with different phone numbers, depending on which combinations the users chose
- Likewise, each phone could be associated with different emails, depending on which combinations the users chose

### Applying Network Analysis
First, we set up each email, phone, and order ID as nodes. This enabled us to model the relationships between emails, phones, and order IDs.

Second, we used the `networkx.connected_components` method to identify all self-contained subgraphs. Intuitively, each self-contained subgraph represents one user. 

Finally, with some data manipulation, we prepared the submission in the requested format.

## Result
We attained the maximum score for this challenge.