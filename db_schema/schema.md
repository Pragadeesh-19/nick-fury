# Therapy Journal DynamoDB Schema

## Tables

### 1. Users Table
**Table Name:** `Users`  
**Primary Key:** `userId` (Partition Key)  
**Attributes:** 
- `userId` (string) - unique user Id
- `userType` (enum) - "client or therapist
- `email` (string) - "users email address"
- `name` (string) - "Full name"
- `gender` (String) - "Clients gender (optional)"
- `location` (string) - "Therapist's location".
- `expertise` (string) - "Therapist's specialization".
- `createdAt` (LocalDateTime) - Timestamp. 

---

### 2. Sessions Table
**Table Name:** `Sessions`  
**Primary Key:**  
- Partition Key: `therapistId`  
- Sort Key: `sessionId`  
**Attributes:**
- `therapistId` (string) - Therapist owner of session
- `sessionId` (string) - unique sessionId. 
- `startTime` (LocalDate) - start time
- `endTime` (LocalDate) - end time. 
- `status` (enum) - Available/booked/cancelled. 
- `privateNotes` (string) - Therapist only notes
- `sharedNotes` (string) - Client-therapist notes.

---

### 3. Appointments Table
**Table Name:** `Appointments`  
**Primary Key:** `appointmentId` (Partition Key)  
**Attributes:**
- `appointmentId` (string) - unique Id
- `sessionId` (string) - Linked sessionId. 
- `clientId` (string) - Client Id
- `therapistId` (string) - therapist Id. 
- `status` (enum) - requested/confirmed/cancelled.
- `mappingId` (string) - created when status is confirmed. 
- `requestedAt` (LocalDateTime) - timestamp  

---

### 4. Mappings Table
**Table Name:** `Mappings`  
**Primary Key:** `mappingId` (Partition Key)  
**Attributes:**
- `mappingId` (string) - Unique mappingId.
- `clientId` (string) - Client Id. 
- `therapistId` (string) - therapist Id. 
- `journalAccess` (boolean) - Access granted flag (true/false)
- `journalAccessRequested` (boolean) - Access granted flag. 
- `status` (enum) - pending/active/inactive.
- `lastUpdated` (LocalDateTime) - timestamp 

---

### 5. Journal Entries Table
**Table Name:** `JournalEntries`  
**Primary Key:**  
- Partition Key: `clientId`  
- Sort Key: `entryId`  
**Attributes:**
- `clientId` (string) - Client Id. 
- `entryId` (string) - unique entry Id. 
- `timestamp` (localdateTime) - timestamp.
- `feeling` (string) - Emotional state. 
- `intensity` (Integer) - 1 to 10 scale. 
- `notes` (string) - detailed journal entry.  

---

### 6. Messages Table
**Table Name:** `Messages`  
**Primary Key:** `messageId` (Partition Key) 
**Attributes:**
- `messageId` (string) - unique messageId. 
- `senderId` (string) - userId of sender. 
- `receiverId` (string) - userId of receiver.
- `content` (string) - Message text. 
- `timestamp` (LocalDateTime) - timestamp

---