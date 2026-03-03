# Stock Take Android Application

**Single app** for TV Sales & Home stock-taking. Launcher icon: TV SALES & HOME logo (red circle with TV, grey ring).

A complete Android stock-taking application built with Kotlin and SQLite. Imports CSV/Excel, drawer navigation, activity log, and variance reports.

---

### 1. No third-party plugins

All features use only the **standard Android and Kotlin toolchain**:

- **Build**: Android Gradle Plugin (`com.android.application`) and Kotlin Gradle plugin (`org.jetbrains.kotlin.android`) — no extra plugins.
- **App**: AndroidX (Core, AppCompat, Material, ConstraintLayout, RecyclerView, Lifecycle, Fragment), Kotlin stdlib, and Coroutines. **Excel (.xlsx)** parsing uses **Apache POI**; CSV can be handled with native Kotlin/Java or POI depending on configuration.
- **Parsing**: CSV and XLSX import; Excel reading via Apache POI for .xlsx files.

### 2. OpenBravo file support

The app reads **OpenBravo-style exports** such as:

- `ReportWarehousePartnerJR.jrxml-_31_.csv` (and any `.csv` with similar names)
- Any CSV/Excel where the **Article** column contains combined values like `OB-MP-GB-711860516-B/R/S LUXURY 4PCE`

**Behaviour:**

- **Column detection**: Headers like `Article`, `Item Code`, `Quantity`, `Quantity On Hand`, `Warehouse Quantity`, etc. are recognised.
- **Article column**: If the first data column is “Article” (or similar), the parser **splits** it into:
  - **Item code**: e.g. `OB-MP-GB-711860516`
  - **Description**: e.g. `B/R/S LUXURY 4PCE`
- **2-column CSV**: If the file has only two columns (e.g. Article + Quantity), the app treats column 1 as quantity and splits the Article column as above.
- **3+ columns**: Item code, description, and quantity are detected from header names; combined Article values are still split when applicable.

### 3. Gradle 9.3.1 (latest as of mid-February 2026)

The project is set up to run with **Gradle 9.3.1** and **Android Gradle Plugin (AGP) 9.0.1**:

- `gradle/wrapper/gradle-wrapper.properties` uses **Gradle 9.3.1**.
- Root `build.gradle` declares **AGP 9.0.1** and **Kotlin 2.0.21** with `apply false`; the app module applies them without specifying versions.
- `settings.gradle` uses `pluginManagement` and `dependencyResolutionManagement` so plugin and dependency versions resolve correctly.

**To build:** Use Android Studio (which will use the wrapper), or run `./gradlew assembleDebug` (or `gradlew.bat` on Windows) after the Gradle wrapper is generated or restored.

## Features

- ✅ **Excel File Import**: Select and import `.xlsx` files from device storage using Storage Access Framework
- ✅ **Data Cleaning**: Automatically splits "Article" column at first "/" into `item_code` and `description`
- ✅ **Local SQLite Database**: All data stored locally, works completely offline
- ✅ **Stock Taking**: Edit counted quantities and automatically calculate variance
- ✅ **Visual Feedback**: Green highlight for zero variance, red highlight for non-zero variance
- ✅ **RecyclerView**: Efficient list display with proper data binding

## Project Structure

```
StockTakeApp/
├── app/
│   ├── build.gradle                    # App-level Gradle config with Apache POI
│   └── src/main/
│       ├── AndroidManifest.xml         # App manifest
│       ├── java/com/example/stocktake/
│       │   ├── StockTakeApplication.kt # Application class
│       │   ├── domain/
│       │   │   └── StockItem.kt        # Data model
│       │   ├── data/
│       │   │   ├── StockDatabaseHelper.kt  # SQLite helper
│       │   │   ├── ExcelReader.kt          # Apache POI Excel reader
│       │   │   └── StockRepository.kt      # Repository pattern
│       │   └── ui/
│       │       ├── MainActivity.kt         # Main activity with file picker
│       │       ├── StockViewModel.kt       # ViewModel
│       │       └── StockAdapter.kt         # RecyclerView adapter
│       └── res/layout/
│           ├── activity_main.xml       # Main activity layout
│           └── item_stock.xml          # RecyclerView item layout
├── build.gradle                        # Project-level Gradle config
└── settings.gradle                     # Project settings
```

## Setup Instructions

### 1. Create New Android Studio Project

1. Open Android Studio
2. Create a new project:
   - **Template**: Empty Activity
   - **Name**: StockTakeApp
   - **Package**: `com.example.stocktake`
   - **Language**: Kotlin
   - **Minimum SDK**: API 24 (Android 7.0)

### 2. Replace Gradle Files

Replace the auto-generated Gradle files with the provided ones:

- `settings.gradle`
- `build.gradle` (root)
- `app/build.gradle`

### 3. Create Package Structure

Create the following packages under `com.example.stocktake`:

- `domain`
- `data`
- `ui`

### 4. Add Source Files

Copy all the Kotlin source files to their respective packages:

- `StockTakeApplication.kt` → root package
- `domain/StockItem.kt`
- `data/StockDatabaseHelper.kt`
- `data/ExcelReader.kt`
- `data/StockRepository.kt`
- `ui/MainActivity.kt`
- `ui/StockViewModel.kt`
- `ui/StockAdapter.kt`

### 5. Add Layout Files

Copy the XML layout files to `app/src/main/res/layout/`:

- `activity_main.xml`
- `item_stock.xml`

### 6. Update AndroidManifest.xml

Replace the manifest with the provided one (or ensure it matches).

### 7. Sync Gradle

Click **Sync Project with Gradle Files** to download dependencies (Apache POI, etc.).

## Excel File Format

Your Excel file should have:

1. **Header Row** (first row) with columns:

   - `Article` - Contains format: `OB-MP-GB-711890074-B/SET INSTINCT 1830MM KING`
   - `Quantity` (or `system_quantity` or `System Quantity`) - Contains the system quantity
2. **Data Rows**: Each row represents one stock item

### Data Cleaning Logic

The app automatically processes the `Article` column:

- **Before first "/"** → `item_code` (e.g., `OB-MP-GB-711890074-B`)
- **After first "/"** → `description` (e.g., `SET INSTINCT 1830MM KING`)

### Configuring Column Names

If your Excel file uses different column names, edit `ExcelReader.kt`:

```kotlin
// In ExcelReader.kt, update these constants:
const val COLUMN_ARTICLE = "Article"  // Change if your column has a different name
const val COLUMN_QUANTITY = "Quantity"  // Change to match your quantity column name
```

The code also checks for alternative names:

- `Quantity`, `system_quantity`, `System Quantity`

## How to Use

1. **Launch the App**: Run the app on an emulator or device
2. **Import Excel File**:

   - Tap the **"Import Excel File"** button
   - Select an `.xlsx` file from your device storage
   - The app will:
     - Read the Excel file
     - Split the Article column
     - Import all items into SQLite
     - Display them in the list
3. **Enter Counted Quantities**:

   - Scroll through the list
   - Enter counted quantities in the **Counted** field
   - Variance is calculated automatically: `variance = counted_quantity - system_quantity`
4. **Visual Indicators**:

   - **Green background**: Variance = 0 (items match)
   - **Red background**: Variance ≠ 0 (discrepancy found)

## Database Schema

The app creates a SQLite table `stock_items`:

```sql
CREATE TABLE stock_items (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    item_code TEXT NOT NULL,
    description TEXT NOT NULL,
    system_quantity INTEGER NOT NULL,
    counted_quantity INTEGER DEFAULT 0,
    variance INTEGER DEFAULT 0
);
```

## Key Technologies

- **Kotlin**: Modern Android development language
- **SQLite**: Local database storage
- **Apache POI**: Excel file reading (.xlsx)
- **Storage Access Framework**: Modern file picker (no permissions needed)
- **RecyclerView**: Efficient list display
- **ViewModel + LiveData**: MVVM architecture
- **Coroutines**: Asynchronous operations
- **ViewBinding**: Type-safe view references

## Error Handling

The app includes error handling for:

- Invalid Excel file format
- Missing columns
- File read errors
- Database operations

Error messages are displayed via Toast notifications.

## Notes

- **No Permissions Required**: Uses Storage Access Framework (ACTION_OPEN_DOCUMENT), so no storage permissions needed
- **Offline First**: All data stored locally in SQLite
- **Large Files**: Apache POI handles large Excel files efficiently
- **Data Persistence**: Data persists between app sessions

## Troubleshooting

### Excel file not importing

- Check that your Excel file has `Article` and `Quantity` columns
- Ensure the file is a valid `.xlsx` format
- Check Logcat for error messages

### Column not found error

- Verify your Excel column names match the expected names
- Update `COLUMN_ARTICLE` and `COLUMN_QUANTITY` in `ExcelReader.kt` if needed

### App crashes on import

- Check Logcat for stack traces
- Ensure Apache POI dependencies are properly synced
- Verify the Excel file is not corrupted

## License

This project is provided as-is for educational and development purposes.
