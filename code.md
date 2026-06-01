## Session 3: Hands-on — Add Relationships to Library Model (12:00 - 13:00)

### Let's Upgrade Our Library Model!

We'll take yesterday's Library Management entities and add proper relationships.

#### Step 1: Open Your Library Project

```bash
cd ~/cap-training/library-management
code .
```

#### Step 2: Update `db/schema.cds` with Associations

Replace the entire content with:

```cds
namespace lib.management;

// ─── REUSABLE TYPES ─────────────────────────────
type Name    : String(100);
type Email   : String(255);
type Phone   : String(20);
type Amount  : Decimal(8,2);

type BorrowStatus : String(20) enum {
  Borrowed;
  Returned;
  Overdue;
}

// ─── AUTHORS ─────────────────────────────────────
entity Authors {
  key ID        : UUID;
  firstName     : String(50);
  lastName      : String(50);
  nationality   : String(50);
  birthDate     : Date;
  biography     : String(1000);
  email         : Email;
  isActive      : Boolean;
  // One author can write MANY books:
  books         : Association to many Books on books.author = $self;
}

// ─── GENRES ──────────────────────────────────────
entity Genres {
  key code      : String(20);
  name          : String(50);
  description   : String(200);
  isActive      : Boolean;
}

// ─── BOOKS ───────────────────────────────────────
entity Books {
  key ID          : UUID;
  title           : String(200);
  isbn            : String(13);
  pages           : Integer;
  price           : Amount;
  publishedDate   : Date;
  language        : String(30);
  edition         : Integer;
  totalCopies     : Integer;
  availableCopies : Integer;
  summary         : String(2000);
  // Managed associations:
  author          : Association to Authors;       // Many books → one author
  genre           : Association to Genres;        // Many books → one genre
  // One book can have MANY reviews:
  reviews         : Association to many Reviews on reviews.book = $self;
}

// ─── MEMBERS ─────────────────────────────────────
entity Members {
  key ID          : UUID;
  memberNumber    : String(10);
  firstName       : String(50);
  lastName        : String(50);
  email           : Email;
  phone           : Phone;
  address         : String(200);
  joinDate        : Date;
  memberType      : String(20);
  maxBooks        : Integer;
  isActive        : Boolean;
  // One member can have MANY borrowings:
  borrowings      : Association to many Borrowings on borrowings.member = $self;
}

// ─── REVIEWS ─────────────────────────────────────
entity Reviews {
  key ID        : UUID;
  book          : Association to Books;       // This review is for WHICH book
  member        : Association to Members;     // WHO wrote this review
  rating        : Integer;
  comment       : String(500);
  reviewDate    : Date;
}

// ─── BORROWINGS ──────────────────────────────────
entity Borrowings {
  key ID          : UUID;
  member          : Association to Members;   // WHO borrowed
  book            : Association to Books;     // WHAT was borrowed
  borrowDate      : Date;
  dueDate         : Date;
  returnDate      : Date;
  status          : BorrowStatus;
  fineAmount      : Amount;
}
```

---

#### Step 3: Update the Service

Update `srv/library-service.cds`:

```cds
using lib.management from '../db/schema';

service CatalogService {
  entity Books      as projection on management.Books;
  entity Authors    as projection on management.Authors;
  entity Genres     as projection on management.Genres;
  entity Members    as projection on management.Members;
  entity Borrowings as projection on management.Borrowings;
  entity Reviews    as projection on management.Reviews;
}
```

#### Step 4: Update CSV Data

Update `db/data/lib.management-Books.csv` to include author and genre references:

```csv
ID,title,isbn,pages,price,publishedDate,language,edition,totalCopies,availableCopies,summary,author_ID,genre_code
1a2b3c4d-1111-1111-1111-111111111111,Clean Code,9780132350884,464,450.00,2008-08-01,English,1,5,3,A handbook of agile software craftsmanship,a1a1a1a1-aaaa-aaaa-aaaa-aaaaaaaaaaaa,PROG
5ed1c590-e7bc-4dcb-a1f3-220892265591,CAP BTP Bookk,9780132350883,500,700,2008-08-05,English,1,5,3,CAP BTP Programming Knowldge,a1a1a1a1-aaaa-aaaa-aaaa-aaaaaaaaaaaa,SCI
2a2b3c4d-2222-2222-2222-222222222222,The Pragmatic Programmer,9780135957059,352,550.00,2019-09-13,English,2,3,2,Journey to mastery,b2b2b2b2-bbbb-bbbb-bbbb-bbbbbbbbbbbb,PROG
3a2b3c4d-3333-3333-3333-333333333333,Sapiens,9780062316097,443,400.00,2015-02-10,English,1,4,4,A brief history of humankind,c3c3c3c3-cccc-cccc-cccc-cccccccccccc,HIST
4a2b3c4d-4444-4444-4444-444444444444,Atomic Habits,9780735211292,320,350.00,2018-10-16,English,1,6,5,Tiny changes remarkable results,d4d4d4d4-dddd-dddd-dddd-dddddddddddd,SELF
5a2b3c4d-5555-5555-5555-555555555555,Foundation,9780553293357,255,300.00,1951-06-01,English,1,5,4,Classic science fiction novel,e5e5e5e5-eeee-eeee-eeee-eeeeeeeeeeee,SCI
```

Update `db/data/lib.management-Authors.csv`:

```csv
ID,firstName,lastName,nationality,birthDate,biography,email,isActive
a1a1a1a1-aaaa-aaaa-aaaa-aaaaaaaaaaaa,Robert,Martin,USA,1952-12-05,Clean Code author and software consultant,unclebob@example.com,true
b2b2b2b2-bbbb-bbbb-bbbb-bbbbbbbbbbbb,David,Thomas,USA,1956-01-01,Co-author of The Pragmatic Programmer,david.thomas@example.com,true
c3c3c3c3-cccc-cccc-cccc-cccccccccccc,Yuval,Harari,Israel,1976-02-24,Historian and philosopher,yuval@example.com,true
d4d4d4d4-dddd-dddd-dddd-dddddddddddd,James,Clear,USA,1986-01-22,Author of Atomic Habits,james.clear@example.com,true
e5e5e5e5-eeee-eeee-eeee-eeeeeeeeeeee,Isaac,Asimov,Russia,1920-01-02,Famous science fiction author,asimov@example.com,true
```

Update `db/data/lib.management-Genres.csv`:

```csv
code,name,description,isActive
PROG,Programming,Software development and coding books,true
HIST,History,Historical and civilization books,true
SELF,Self Help,Personal growth and productivity,true
FICT,Fiction,Novels and fictional stories,true
SCI,Science,Scientific discoveries and research,true
```

#### Step 5: Run and Test with $expand

```bash
cds watch
```

Now try these powerful queries:

```http
// Get books WITH author details:
GET http://localhost:4004/Catalog/Books?$expand=author

// Get books WITH genre details:
GET http://localhost:4004/Catalog/Books?$expand=genre

// Get books with BOTH author and genre:
GET http://localhost:4004/Catalog/Books?$expand=author,genre

// Get an author WITH all their books:
GET http://localhost:4004/Catalog/Authors?$expand=books

// Get borrowings with book and member details:
GET http://localhost:4004/Catalog/Borrowings?$expand=book,member

// Get members with their borrowings AND the borrowed book:
GET http://localhost:4004/Catalog/Members?$expand=borrowings($expand=book)

// Filter books by a specific author:
GET http://localhost:4004/Catalog/Books?$filter=author_ID eq a1a1a1a1-aaaa-aaaa-aaaa-aaaaaaaaaaaa
```
