## Session 3: Hands-on — Build Views & Projections (12:00 - 13:00)

### Exercise: Add Views to Your Library Project

Open your Library Management project from Day 13.

#### Step 1: Create a New Views File

Create `db/views.cds`:

```cds
using lib.management from './schema';

// ─── VIEW 1: Available Books (for library kiosk) ───
entity AvailableBooks as select from management.Books {
  ID,
  title,
  isbn,
  price,
  availableCopies,
  author.firstName || ' ' || author.lastName as authorName : String(101),
  genre.name as genreName
} where availableCopies > 0;

// ─── VIEW 2: Overdue Borrowings (for staff) ────────
entity OverdueBorrowings as select from management.Borrowings {
  ID,
  borrowDate,
  dueDate,
  fineAmount,
  book.title       as bookTitle,
  member.firstName || ' ' || member.lastName as memberName : String(101),
  member.email     as memberEmail,
  member.phone     as memberPhone
} where status = 'Overdue';

// ─── VIEW 3: Book Summary with pricing ─────────────
entity BookPricing as select from management.Books {
  ID,
  title,
  price,
  (price * 0.9)  as memberPrice  : Decimal(8,2),
  (price * 0.18) as gstAmount    : Decimal(8,2),
  (price * 1.18) as priceWithGST : Decimal(8,2),
  case
    when price > 500 then 'Premium'
    when price > 300 then 'Standard'
    else 'Budget'
  end as priceCategory : String(20)
};

// ─── VIEW 4: Member Activity ───────────────────────
entity MemberActivity as select from management.Members {
  ID,
  memberNumber,
  firstName || ' ' || lastName as fullName : String(101),
  email,
  memberType,
  maxBooks,
  isActive
};
```

---

#### Step 2: Expose Views in a Separate Reporting Service

Create `srv/reporting-service.cds`:

```cds
using from '../db/views';

service ReportingService @(path: '/reports') {
  @readonly entity AvailableBooks   as projection on AvailableBooks;
  @readonly entity OverdueBorrowings as projection on OverdueBorrowings;
  @readonly entity BookPricing      as projection on BookPricing;
  @readonly entity MemberActivity   as projection on MemberActivity;
}
```

#### Step 3: Update Main Service with Projections

Update `srv/library-service.cds` to use exclusions:

```cds
using lib.management from '../db/schema';

service LibraryService @(path: '/library') {
  // Full CRUD:
  entity Books      as projection on management.Books;
  entity Authors    as projection on management.Authors;
  entity Genres     as projection on management.Genres;
  entity Members    as projection on management.Members;
  entity Borrowings as projection on management.Borrowings;
  entity Reviews    as projection on management.Reviews;
  entity PurchaseOrders as projection on management.PurchaseOrders;
}
```

#### Step 4: Test

```bash
cds watch
```

Test the reporting service:

```http
// Available books with calculated fields:
GET http://localhost:4004/reports/AvailableBooks

// Overdue borrowings with member contact info:
GET http://localhost:4004/reports/OverdueBorrowings

// Book pricing with GST calculations:
GET http://localhost:4004/reports/BookPricing

// Filter premium books only:
GET http://localhost:4004/reports/BookPricing?$filter=priceCategory eq 'Premium'

// Member activity:
GET http://localhost:4004/reports/MemberActivity?$filter=isActive eq true
```
