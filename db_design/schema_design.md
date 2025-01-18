# Database Schema for Therapy Journal Application

## 1. Users Table (`UsersTable`)
### Description:
Stores information about both clients and therapists.

### Attributes:
- `userId` (String): Unique identifier for each user (HASH Partition Key).
- `email` (String): Email address of the user.
- `password` (String): Password hash of the user.
- `userType` (String): Type of the user, either `CLIENT` or `THERAPIST`.

### Primary Keys:
- Hash Partition Key: `userId`

### Secondary Indexes:
- None

## 2. Relationships Table (`RelationshipsTable`)
### Description:
Stores the mapping between clients and therapists, including journal access status.

### Attributes:
-   `relationshipId` (String): Unique identifier for each relationship (HASH Partition Key).
-   `clientId` (String): ID of the client.
-   `therapistId` (String): ID of the therapist.
-   `hasJournalAccess` (Boolean): Indicates if the therapist has journal access for the client.

### Primary Keys:
- Hash Partition Key: `relationshipId`

### Secondary Indexes:
-   GSI : `TherapistClientIndex`
    *   Partition Key: `therapistId`
    *   Sort Key: `clientId`

## 3. Messages Table (`MessagesTable`)
### Description:
Stores messages exchanged between clients and therapists.

### Attributes:
-   `messageId` (String): Unique identifier for each message (HASH Partition Key).
-   `senderId` (String): ID of the message sender.
-   `receiverId` (String): ID of the message receiver.
-   `message` (String): The message content.
-   `timestamp` (String): Message timestamp in `YYYY-MM-DDTHH:MM:SSZ` format.

### Primary Keys:
-   Hash Partition Key: `messageId`

### Secondary Indexes:
-   GSI: `SenderReceiverIndex`
    *   Partition Key: `senderId`
    *  Sort Key: `receiverId`

## 4. Emotion Journals Table (`EmotionJournalsTable`)
### Description:
Stores emotion records for clients.

### Attributes:
-   `emotionId` (String): Unique identifier for each emotion record (HASH Partition Key).
-   `clientId` (String): ID of the client.
-   `timeOfEmotion` (String): Timestamp of the emotion in `YYYY-MM-DDTHH:MM:SSZ` format.
-   `feeling` (String): Name of the feeling or emotion.
-   `intensity` (String): Intensity of the emotion (e.g., "LOW", "MEDIUM", "HIGH").

### Primary Keys:
-   Hash Partition Key: `emotionId`

### Secondary Indexes:
-   GSI: `ClientTimeIndex`
    *   Partition Key: `clientId`
    *   Sort Key: `timeOfEmotion`

## 5. Appointments Table (`AppointmentsTable`)
### Description:
Stores appointment details.

### Attributes:
-   `appointmentId` (String): Unique identifier for each appointment (HASH Partition Key).
-   `therapistId` (String): ID of the therapist.
-   `clientId` (String): ID of the client.
-   `appointmentTime` (String): Appointment time in `YYYY-MM-DDTHH:MM:SSZ` format.
-   `status` (String): Appointment status (`PENDING`, `CONFIRMED`, or `CANCELLED`).

### Primary Keys:
-   Hash Partition Key: `appointmentId`

### Secondary Indexes:
- GSI: `TherapistTimeIndex`
    *  Partition Key: `therapistId`
    * Sort Key: `appointmentTime`
-   GSI: `ClientTimeIndex`
    *   Partition Key: `clientId`
    *   Sort Key: `appointmentTime`

## 6. Sessions Table (`SessionsTable`)
### Description:
Stores therapy session details.

### Attributes:
-   `sessionId` (String): Unique identifier for each session (HASH Partition Key).
-   `therapistId` (String): ID of the therapist.
-  `clientId` (String): ID of the client.
-   `sessionTime` (String): Session time in `YYYY-MM-DDTHH:MM:SSZ` format.
-   `privateNotes` (String): Private notes of the therapist.
- `sharedNotes` (String): Shared notes of the therapist

### Primary Keys:
-   Hash Partition Key: `sessionId`

### Secondary Indexes:
-   GSI: `TherapistClientTimeIndex`
    *   Partition Key: `therapistId`
    *   Sort Key: `clientId`, `sessionTime`


## API Query Construction and GSI Usage:

Here's how the APIs would use this schema:
### Authentication
- /auth/register:  `UsersTable` is used to insert a new record, using email, password and userType as a parameter in the request body.
- /auth/login: `UsersTable` is used to fetch record using email and password as a parameter, using the `userId` as the partition key.

### Relationships
-   `GET /relationships` (get list of relationships):
    *   Uses `TherapistClientIndex` with `therapistId` or `clientId` or both provided in the query parameters to fetch the relationship records based on user type.
-   `POST /relationships` (map therapist to client):
    *   `RelationshipsTable` is used with a generated `relationshipId`
-   `DELETE /relationships` (remove a relationship):
    *   `RelationshipsTable` is used with a given  `relationshipId`.
-  `POST /relationships/journalAccess` (request a new client for journal access):
      *  `RelationshipsTable` is used with generated `relationshipId`
-  `DELETE /relationships/journalAccess` (remove journal access):
       *  `RelationshipsTable` is used with  the `therapistId` and `clientId` to delete the record, query the `TherapistClientIndex` to fetch and delete record based on `clientId` and `therapistId`.
-   `GET /relationships/journalAccessRequests` (get list of journal access request):
    *    `RelationshipsTable` is used with `clientId` in the query parameters, by querying the `TherapistClientIndex` with the `clientId`.
-   `PUT /relationships/journalAccessRequests` (approve/reject journal access request):
        *  `RelationshipsTable` is used with  `clientId` and `therapistId` to find and update the record, query the `TherapistClientIndex` using `therapistId` and `clientId`.

### Messages
-   `POST /messages` (send a message):
    *   `MessagesTable` is used with generated `messageId`.
-   `GET /messages` (get message history):
    *   `MessagesTable` is used with `senderId` and `receiverId` using the `SenderReceiverIndex` GSI

### Emotion Journal
- `POST /emotion-journal` (add a new emotion record):
    *  `EmotionJournalsTable` is used with a generated `emotionId`.
- `GET /emotion-journal` (retrieve emotion records):
    * `EmotionJournalsTable` is used using the `ClientTimeIndex` GSI using the query parameter `clientId`.

### Appointments
-  `POST /appointments` (request an appointment):
     * `AppointmentsTable` is used with a generated `appointmentId`.
-   `GET /appointments` (get appointments for a client):
        *  `AppointmentsTable` is used with the `ClientTimeIndex` using the `clientId` as the query parameters.
-  `PUT /appointments/{appointmentId}` (update appointment status):
    * `AppointmentsTable` is used with the  `appointmentId` as primary key
-   `GET /therapists/{therapistId}/appointments` (get appointments for a therapist):
    *    `AppointmentsTable` is used with `TherapistTimeIndex` using `therapistId` as partition key and `appointmentTime` as sort key.

### Therapists
-  `GET /therapists` (search and select for a list of therapists)
   *  `UsersTable` is used with `userType` as THERAPIST and then filter with the keyword if provided using the search functionality.

### Search
 -   `GET /search` (search for notes and emotions journals)
    *   For a Client: Scan the `EmotionJournalsTable` using the `keyword` parameter on the `feeling` attribute.
    *   For a Therapist: Scan the `SessionsTable` using the `keyword` parameter on `privateNotes` and `sharedNotes` attribute.