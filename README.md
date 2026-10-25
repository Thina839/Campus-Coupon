# Campus Coupon – starter code (ITC327W, Prominent Programmers)

Built from the Phase 2 design pack: Use Case Diagram, Sequence Diagrams 1 & 2,
System Architecture Diagram, ERD, 9 Flutter screens and 8 ASP.NET screens.

```
campus-coupon/
├── supabase/        Database (ERD) + business rules + security  -> run in Supabase SQL Editor
│   ├── SETUP_ALL.sql      ONE paste = wipe old database + 01,02,03,05,06,07
│   ├── 00_reset.sql       wipe only (all data + logins) - used by SETUP_ALL
│   ├── 01_schema.sql      tables from the ERD (+ fields the screens need)
│   ├── 02_functions.sql   RSVP, attendance -> points, redeem, accept, dashboards
│   ├── 03_rls.sql         Row Level Security (who can see/change what)
│   ├── 04_seed.sql        test events + coupons from the Figma screens
│   ├── 05_storage.sql     "event-media" bucket for posters
│   ├── 06_cafeteria_staff.sql  Cafeteria Staff: register, approve/reject (+refund), history
│   └── 07_meals.sql       Meals/Coupons page: who may add/edit meals, photo uploads
├── flutter_app/     Student mobile app (Sequence Diagram 1)
└── admin_portal/    ASP.NET Core MVC web portal: Event Administrator + Cafeteria Staff (Sequence Diagram 2)
```

## Quick start
1. **Supabase, clean start**: SQL Editor → paste and run **`SETUP_ALL.sql`** (deletes the old data and logins!),
   register an admin in the portal, confirm the email, then run `04_seed.sql`.
   **Or file by file**: SQL Editor → run `01` → `02` → `03` → `05` → `06` → `07`. Register an admin in the portal, then run `04_seed.sql`.
   *Already ran 01–06 before?* Just run **`07_meals.sql`** once. It keeps all existing data.
2. **Admin / Cafeteria portal** (Visual Studio 2022 17.8+, .NET 8): double-click `CampusCoupon.sln`, press F5.
   `appsettings.json` already holds the project URL and publishable key.
3. **Flutter app** (Flutter 3.22+): `lib/config.dart` already holds the same URL and key.
   ```
   cd flutter_app
   flutter create . --project-name campus_coupon --platforms=android,web
   flutter pub get
   flutter run
   ```
   Then add `<uses-permission android:name="android.permission.INTERNET"/>` to
   `android/app/src/main/AndroidManifest.xml` before building the APK.

## The three users (Use Case Diagram)
| User | App | Can do |
|---|---|---|
| Student | Flutter app | Register, view events, RSVP, wallet, transaction history, redeem coupon |
| Event Administrator | Web portal | Dashboard, create/update/publish events, RSVP list, record attendance, **Coupons** (all Food + Retail), view redemptions |
| Cafeteria Staff | Web portal | **Current Redemptions**: approve (give meal) or reject (no meal, points refunded); **Redemption History** with search, dates and CSV download; **Meals**: add, edit, price, photo, switch on/off |

Staff register at **Register → Account type** (Event Administrator / Cafeteria Staff), with a
`@cut.ac.za` email and a unique staff number, then log in with staff number or email.
A cafeteria login only ever sees the two cafeteria pages.

## Test status
- `06_cafeteria_staff.sql` was run twice on a copy of a live-style database (PostgreSQL 16) and passed 20 role-based tests:
  staff registration rules, login by staff number, approve, reject + refund, no double use, expired coupons, students/admins blocked.
- `supabase/*.sql` 01–04 were executed and tested on PostgreSQL 16 with a mock of Supabase's `auth` schema:
  registration rules, RSVP duplicates, attendance → points exactly once, no points when absent,
  insufficient-balance rejection, redemption codes, one-time acceptance and RLS isolation all passed.
  `05_storage.sql` uses Supabase's `storage` schema and was not run locally.
- The Flutter and ASP.NET code was written to the current APIs but **not compiled** (no SDK available
  in the build environment). Run `flutter analyze` and `dotnet build` first and fix anything they report.

Never commit the Supabase **service_role** key. The anon key is safe in the app only because RLS is on.
