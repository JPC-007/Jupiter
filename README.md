# Jupiter

A personalized event recommendation web application that helps users discover nearby events based on their location and preferences.

## Features

- **Event Search**: Search for events near your location using the TicketMaster API
- **Personalized Recommendations**: Get event suggestions based on your favorite categories
- **Favorites Management**: Save and manage your favorite events
- **Geo-based Discovery**: Find events within a 50-mile radius of your location
- **Category Filtering**: Events are categorized (Music, Sports, Arts, etc.) for easy browsing

## Tech Stack

### Backend
- **Java Servlet** - RESTful API endpoints
- **MySQL** - Data persistence for users, items, and favorites
- **TicketMaster Discovery API** - Real-time event data source

### Frontend
- **HTML5 / CSS3** - Responsive UI design
- **JavaScript** - Dynamic content loading and user interactions
- **Font Awesome** - Icon library

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend                              │
│                   (HTML/CSS/JavaScript)                      │
└─────────────────────────┬───────────────────────────────────┘
                          │ HTTP/JSON
┌─────────────────────────▼───────────────────────────────────┐
│                     RPC Layer (Servlets)                     │
│         SearchItem | ItemHistory | RecommendItem             │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                    Business Logic                            │
│              GeoRecommendation Algorithm                     │
└──────────┬──────────────────────────────────┬───────────────┘
           │                                  │
┌──────────▼──────────┐            ┌──────────▼──────────┐
│   External API      │            │   Database Layer    │
│  TicketMaster API   │            │  MySQL Connection   │
└─────────────────────┘            └─────────────────────┘
```

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/search` | GET | Search events by location and keyword |
| `/history` | GET/POST/DELETE | Manage user favorite events |
| `/recommendation` | GET | Get personalized event recommendations |

## Database Schema

```sql
-- Users table
users (user_id, password, first_name, last_name)

-- Events/Items table
items (item_id, name, rating, address, image_url, url, distance)

-- Event categories
categories (item_id, category)

-- User favorites history
history (user_id, item_id, last_favor_time)
```

## Getting Started

### Prerequisites

- JDK 8 or higher
- Apache Tomcat 9.x
- MySQL 8.0
- Eclipse IDE (optional, for development)

### Database Setup

1. Create MySQL database:
```sql
CREATE DATABASE laiproject;
```

2. Update database credentials in `src/main/java/db/mysql/MySQLDBUtil.java`:
```java
private static final String HOSTNAME = "localhost";
private static final String PORT_NUM = "3306";
private static final String USERNAME = "your_username";
private static final String PASSWORD = "your_password";
```

3. Run `MySQLTableCreation.java` to initialize tables:
```bash
java -cp .:mysql-connector-java-8.0.11.jar db.mysql.MySQLTableCreation
```

### Deployment

1. Build the project as a WAR file
2. Deploy to Apache Tomcat
3. Access the application at `http://localhost:8080/Jupiter`

### Default Test User

- **User ID**: 1111
- **Password**: 3229c1097c00d497a0fd282d586be050 (MD5 hashed)

## Recommendation Algorithm

The system uses a content-based collaborative filtering approach:

1. **Collect user favorites** - Retrieve all items the user has favorited
2. **Extract categories** - Count category occurrences from favorites
3. **Rank categories** - Sort categories by frequency
4. **Search by category** - Query TicketMaster API for each category
5. **Filter & Sort** - Remove already-favorited items, sort by distance

## Project Structure

```
Jupiter/
├── src/main/java/
│   ├── algorithm/          # Recommendation algorithm
│   ├── db/                 # Database layer
│   │   └── mysql/          # MySQL implementation
│   ├── entity/             # Data models (Item)
│   ├── external/           # External API integration
│   └── rpc/                # Servlet endpoints
├── src/main/webapp/
│   ├── scripts/            # JavaScript files
│   ├── styles/             # CSS files
│   ├── WEB-INF/lib/        # Dependencies (JAR files)
│   └── index.html          # Main page
└── README.md
```

## License

This project is for educational purposes.

## Acknowledgments

- [TicketMaster API](https://developer.ticketmaster.com/) for event data
- [Font Awesome](https://fontawesome.com/) for icons
