

```tasks
tags includes #JobHunt
not done 
```

```dataviewjs
// Collect pages with 'date_applied' and convert them into an array of dates
var pagesArray = Array.from(dv.pages()
    .where(p => p.date_applied)
    .map(p => dv.date(p.date_applied).toFormat("yyyy-MM-dd")));

// Find the earliest and latest date in your dataset
var sortedPagesArray = [...pagesArray].sort();
var startDate = dv.date(sortedPagesArray[0]);
var endDate = dv.date(sortedPagesArray[sortedPagesArray.length - 1]);

// Generate all workdays (Monday to Friday) in the range
var allWorkdays = [];
for (var d = startDate; d <= endDate; d = d.plus({ days: 1 })) {
    if (d.weekday >= 1 && d.weekday <= 5) { // 1 is Monday, 5 is Friday
        allWorkdays.push(d.toFormat("yyyy-MM-dd"));
    }
}

// Count occurrences of each date
var dateCounts = allWorkdays.reduce((acc, date) => {
    acc[date] = (acc[date] || 0); // Initialize all dates with 0
    return acc;
}, {});

// Add counts from your data
pagesArray.forEach(date => {
    dateCounts[date] = (dateCounts[date] || 0) + 1;
});

// Convert the dateCounts object into an array of [date, count] pairs
var dateCountPairs = Object.entries(dateCounts);

// Sort the array by date
dateCountPairs.sort((a, b) => dv.date(a[0]) - dv.date(b[0]));

// Prepare labels and data for the chart
const labels = dateCountPairs.map(pair => pair[0]);
const jobApplications = dateCountPairs.map(pair => pair[1]);

// Chart configuration
const chartData = {
    type: 'bar',
    data: {
        labels: labels,
        datasets: [{
            label: '#jobrec',
            data: jobApplications,
            backgroundColor: 'rgba(54, 162, 235, 0.2)',
            borderColor: 'rgba(54, 162, 235, 1)',
            borderWidth: 1
        }]
    },
    options: {
        scales: {
            y: {
                beginAtZero: true
            }
        }
    }
};

// Render the chart
window.renderChart(chartData, this.container);
```

```dataviewjs
// Collect pages with 'date_applied' and convert them into an array of dates
var pagesArray = Array.from(dv.pages()
    .where(p => p.date_applied)
    .map(p => dv.date(p.date_applied).toFormat("yyyy-MM-dd")));

// Ensure the pagesArray is sorted to correctly identify startDate and endDate
var sortedPagesArray = pagesArray.sort();
var startDate = dv.date(sortedPagesArray[0]); // Make sure this line exists and is correctly placed
var endDate = dv.date(sortedPagesArray[sortedPagesArray.length - 1]); // And this line too

// Function to get the year and week number for a date
function getYearWeek(d) {
    let date = new Date(d);
    date.setHours(0, 0, 0, 0);
    // Thursday in current week decides the year.
    date.setDate(date.getDate() + 3 - (date.getDay() + 6) % 7);
    // January 4 is always in week 1.
    let year = new Date(date.getFullYear(), 0, 4);
    // Adjust to Thursday in week 1 and count number of weeks from date to week1.
    let weekNum = 1 + Math.round(((date - year) / 86400000 - 3 + (year.getDay() + 6) % 7) / 7);
    return `${date.getFullYear()}-${weekNum < 10 ? '0' + weekNum : weekNum}`;
}

// Generate all weeks between startDate and endDate
var allWeeks = {};
for (var d = startDate; d <= endDate; d = d.plus({ days: 1 })) {
    let yearWeek = getYearWeek(d.toISODate());
    if (!allWeeks[yearWeek]) {
        allWeeks[yearWeek] = 0; // Initialize all year-weeks with 0
    }
}

// Add counts from your data to weeks
pagesArray.forEach(date => {
    let yearWeek = getYearWeek(date);
    allWeeks[yearWeek] = (allWeeks[yearWeek] || 0) + 1;
});

// Convert the allWeeks object into an array of [year-week, count] pairs and sort by year-week
var weekCountPairs = Object.entries(allWeeks).sort((a, b) => a[0].localeCompare(b[0]));

// Prepare labels and data for the chart
const labels = weekCountPairs.map(pair => `Week ${pair[0]}`);
const jobApplications = weekCountPairs.map(pair => pair[1]);

// Chart configuration
const chartData = {
    type: 'bar',
    data: {
        labels: labels,
        datasets: [{
            label: '#jobrec',
            data: jobApplications,
            backgroundColor: 'rgba(54, 162, 235, 0.2)',
            borderColor: 'rgba(54, 162, 235, 1)',
            borderWidth: 1
        }]
    },
    options: {
        scales: {
            y: {
                beginAtZero: true
            }
        }
    }
};

// Render the chart
window.renderChart(chartData, this.container);
```
#### In-Flight:
```dataview
table
company as "Company", 
role as "Role",
recruiter_screen as "Phone Screen", 
interview as "Interview", 
dateformat(date_applied, "MM-dd-yyyy") as "Date Applied"
from "Personal/Job Hunt/Companies" 
where contains(file.tags, "#JobPost") 
and recruiter_screen = true
and applied = true
and rejection = false
and declined = false
sort date_applied desc
```


#### Jobs Applied For:

```dataview
table
company as "Company", 
role as "Role",
recruiter_screen as "Phone Screen", 
interview as "Interview", 
rejection as "Rejected", 
dateformat(date_applied, "MM-dd-yyyy") as "Date Applied"
from "Personal/Job Hunt/Companies" 
where contains(file.tags, "#JobPost") 
and applied = true
and rejection = false
and declined = false
sort date_applied asc
```

#### Rejections/Declined:
```dataview
table
company as "Company", 
role as "Role",
recruiter_screen as "Phone Screen", 
interview as "Interview", 
rejection as "Rejected",
declined as "Declined",
dateformat(date_applied, "MM-dd-yyyy") as "Date Applied"
from "Personal/Job Hunt/Companies" 
where contains(file.tags, "#JobPost") 
and applied = true
and rejection = true
or declined = true
sort date_applied desc
```

```tasks
tags includes #JobHunt
done 
```
