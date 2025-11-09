# data-quality-dashboard

Creating a comprehensive data quality dashboard requires integrating several components. Below is an example Python program using the Flask web framework along with Pandas for data processing and Plotly for data visualization. This program sets up a basic web application where users can upload datasets and view basic data quality metrics and visualizations.

To keep this example focused and educational, I'll demonstrate key components such as dataset upload, processing, error handling, and visualization. However, please note that building a production-ready dashboard would involve additional considerations, such as user authentication, more sophisticated error handling, and deployment strategies.

Before running this program, make sure you have the necessary packages installed. You can install them using pip:

```bash
pip install flask pandas plotly dash
```

### File Structure

```
data-quality-dashboard/
│
├── app.py
└── templates/
    └── index.html
```

### app.py

```python
from flask import Flask, request, render_template, redirect, url_for, flash
import pandas as pd
import plotly.express as px
import os

# Initialize Flask app
app = Flask(__name__)
app.secret_key = 'your_secret_key'  # Secret key for session management

ALLOWED_EXTENSIONS = {'csv'}

def allowed_file(filename):
    """Check if the filename extension is allowed."""
    return '.' in filename and filename.rsplit('.', 1)[1].lower() in ALLOWED_EXTENSIONS

def calculate_metrics(df):
    """Calculate basic data quality metrics."""
    metrics = {
        'Missing Values': df.isnull().sum().sum(),
        'Duplicate Rows': df.duplicated().sum(),
        'Total Rows': len(df),
        'Total Columns': len(df.columns),
    }
    return metrics

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        # Check if the post request has the file part
        if 'file' not in request.files:
            flash('No file part')
            return redirect(request.url)

        file = request.files['file']

        # If the user does not select a file, the browser also submits an empty part without a filename
        if file.filename == '':
            flash('No selected file')
            return redirect(request.url)

        if file and allowed_file(file.filename):
            try:
                # Read the dataset into a DataFrame
                df = pd.read_csv(file)
                
                # Calculate data quality metrics
                metrics = calculate_metrics(df)

                # Create a simple visualization
                fig = px.histogram(df, x=df.columns[0], title='Distribution of {}'.format(df.columns[0]))

                # Render the dashboard with metrics and visualization
                return render_template('index.html', metrics=metrics, plot=fig.to_html(full_html=False))

            except Exception as e:
                # Error handling
                flash('An error occurred while processing the file: {}'.format(e))
                return redirect(request.url)

    return render_template('index.html')

if __name__ == '__main__':
    app.run(debug=True)
```

### templates/index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Data Quality Dashboard</title>
</head>
<body>
    <h1>Data Quality Dashboard</h1>
    <form action="/" method="post" enctype="multipart/form-data">
        <input type="file" name="file">
        <input type="submit" value="Upload">
    </form>

    {% with messages = get_flashed_messages() %}
      {% if messages %}
        <ul>
        {% for message in messages %}
          <li style="color: red;">{{ message }}</li>
        {% endfor %}
        </ul>
      {% endif %}
    {% endwith %}

    {% if metrics %}
    <h2>Data Quality Metrics</h2>
    <ul>
        {% for key, value in metrics.items() %}
        <li><strong>{{ key }}:</strong> {{ value }}</li>
        {% endfor %}
    </ul>
    {% endif %}

    {% if plot %}
    <h2>Data Visualization</h2>
    <div>{{ plot|safe }}</div>
    {% endif %}
</body>
</html>
```

### Comments

- **Flask Setup**: The Flask application manages routes and rendering templates.
- **File Handling**: Users can upload CSV files, and the program checks for allowed file types.
- **Error Handling**: Basic error feedback is provided using Flask's `flash` messaging system.
- **Data Processing**: The uploaded dataset is read into a Pandas DataFrame, and basic data quality metrics are calculated.
- **Visualization**: Plotly Express is used to create a simple histogram, which can be replaced or extended with more complex visualizations.

This is a baseline application that can be expanded to include more intricate metrics, interactive visualizations, database integrations, and authentication/authorization for a more comprehensive solution.