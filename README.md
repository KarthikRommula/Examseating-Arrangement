# Exam Seating Arrangement System

A web-based application for generating and managing examination seating arrangements for BTech colleges. It lets you maintain students, examination halls, and exams, then automatically produces a fair, branch- and semester-mixed seating plan for any exam and renders it as a visual per-hall grid.

## Features

- **Student management** — Add students manually or in bulk, with roll number (unique), name, branch, and semester.
- **Hall management** — Define examination halls with a capacity and a grid layout (rows x columns).
- **Exam management** — Create exams with a name, date, and type (`MID` or `SEMESTER`).
- **Automatic seating generation** — Groups students by branch and semester, shuffles each group, interleaves them, and assigns seats across halls while skipping alternate columns to keep same-branch students apart.
- **Visual seating view** — Renders the generated arrangement as a seat grid for each hall, showing each student's roll number, name, branch, and semester.
- **Bulk import** — Import students or halls from CSV or Excel (`.xlsx`/`.xls`) files with header validation and duplicate detection.
- **Template downloads** — Download ready-made Excel templates (with sample rows) for student and hall imports.

## Tech Stack

- **Backend:** Python, [Flask](https://flask.palletsprojects.com/)
- **ORM / Database:** Flask-SQLAlchemy with SQLite
- **Spreadsheet I/O:** openpyxl (Excel), Python `csv` (CSV)
- **Frontend:** Jinja2 server-rendered templates, Bootstrap 5 and Font Awesome 6 (loaded via CDN)

## How It Works

The application is a single Flask app (`app.py`) backed by a SQLite database. Four SQLAlchemy models define the data:

- `Student` — `roll_number`, `name`, `branch`, `semester`
- `Hall` — `name`, `capacity`, `rows`, `columns`
- `Exam` — `name`, `date`, `exam_type` (`MID` / `SEMESTER`)
- `SeatingArrangement` — links an exam, student, and hall to a specific seat (`seat_row`, `seat_column`)

When seating is generated for an exam (`/generate-seating/<exam_id>`):

1. Any existing arrangement for that exam is cleared.
2. Students are grouped by `(branch, semester)` and each group is shuffled.
3. Groups are interleaved into a single mixed list so adjacent students tend to come from different branches/semesters.
4. Students are placed hall by hall. The column index advances by 2 each time (leaving an empty seat between students); when a row fills, it wraps to the next row, and when a hall fills, it moves to the next hall. If halls run out of space, a warning is shown.

The result is persisted as `SeatingArrangement` rows and displayed by `/view_seating/<exam_id>`, which reconstructs a 2D seat grid per hall for rendering.

## Project Structure

```
Examseating-Arrangement/
├── app.py                  # Flask app: models, routes, import/export, seating logic
├── requirements.txt        # Python dependencies
├── instance/
│   └── exam_seating.db     # SQLite database (auto-created on first run)
└── templates/
    ├── base.html           # Shared layout, navbar, Bootstrap/Font Awesome
    ├── index.html          # Home / dashboard
    ├── students.html       # Manage & import students
    ├── halls.html          # Manage halls
    ├── exams.html          # Manage exams, trigger seating generation
    └── seating.html        # Visual seating grid view
```

## Routes

| Method      | Path                          | Description                                          |
|-------------|-------------------------------|------------------------------------------------------|
| GET         | `/`                           | Home dashboard                                       |
| GET, POST   | `/students`                   | List and add students                                |
| GET, POST   | `/halls`                      | List and add halls                                   |
| GET, POST   | `/exams`                      | List and add exams                                   |
| POST        | `/import-data`                | Bulk import students or halls (CSV/Excel)            |
| GET         | `/download-template/<type>`   | Download an Excel import template (`students`/`halls`)|
| GET         | `/generate-seating/<exam_id>` | Generate the seating arrangement for an exam         |
| GET         | `/view_seating/<exam_id>`     | View the seating grid for an exam                    |

## Prerequisites

- Python 3.8 or higher
- `pip`

## Installation

```bash
# Clone the repository
git clone https://github.com/KarthikRommula/Examseating-Arrangement.git
cd Examseating-Arrangement

# (Recommended) create and activate a virtual environment
python -m venv venv
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
```

## Usage

Run the application:

```bash
python app.py
```

The database (`instance/exam_seating.db`) and an `uploads/` folder are created automatically on first run. Open the app at:

```
http://localhost:5000
```

### Typical workflow

1. **Add halls** under *Halls* (manually or by importing).
2. **Add students** under *Students* (manually or by importing CSV/Excel).
3. **Create an exam** under *Exams*.
4. Click **Generate Seating** for that exam, then **View Seating** to see the per-hall grid.

### Bulk import formats

Import expects these column headers (download templates from the app):

- **Students:** `roll_number`, `name`, `branch`, `semester`
- **Halls:** `name`, `capacity`, `rows`, `columns`

Accepted file types: `.csv`, `.xlsx`, `.xls`. Maximum upload size is 16 MB. Rows with an existing unique key (student `roll_number` or hall `name`) are skipped as duplicates.

## Configuration

Configuration is defined in `app.py` via `app.config`:

| Setting                  | Default                       | Description                          |
|--------------------------|-------------------------------|--------------------------------------|
| `SQLALCHEMY_DATABASE_URI`| `sqlite:///exam_seating.db`   | Database connection string           |
| `SECRET_KEY`             | `your-secret-key-here`        | Flask session/flash signing key      |
| `UPLOAD_FOLDER`          | `uploads`                     | Folder for temporary uploads         |
| `MAX_CONTENT_LENGTH`     | `16 * 1024 * 1024` (16 MB)    | Maximum upload size                  |

> Note: For any non-local deployment, replace the default `SECRET_KEY` with a strong, secret value and disable debug mode (`app.run(debug=True)`).
