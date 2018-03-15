
#Colleen Karnas-Haines
#Pandas Homework    3/14/18
#Academy of Pi

#Please see the link to my Tableau visualizations of this dataset

#Observations
#1. The tope schools were Charter schools, the bottom schools were District schools. While the average scores 
#were not very far from each other, the % of students passing did differ significantly from Charter vs District schools
#2. Schools with the most funding per student performed the worst. Charter Schools, with low funding performed better in general.
#I would be interested to understand what was meant by "funding". Charter schools are typically funding at a lower rate *by the state*
#but they do spend a significant amount of time fund-raising and finding alternate forms of funding.
#3. Finally, small schools perform better than larger schools. A great follow-up question to the size issue is how the student-teacher ratio
#changes with the school size. So, in large schools do they have 1,000 extra students per grade, but the same number of 
#students per classroom? Or are those 1,000 extra students shoved into overcrowded classrooms.

```python
import os
import pandas as pd
import numpy as np
```


```python
#Read in files
csv_path_schools = os.path.join("..","Resources","raw_data","schools_complete.csv")
csv_path_students = os.path.join("..","Resources","raw_data","students_complete.csv")

school_data = pd.read_csv(csv_path_schools)
student_data = pd.read_csv(csv_path_students)
```


```python
#Initial test
#school_data.head()
#student_data.head()
#school_data.count()
#student_data.count()
```


```python
#user defined functions
def format_to_nice_number(series_to_change):  
    series_to_change=series_to_change.map("{:,.0f}".format)
    return series_to_change;

def format_to_perc(series_to_change):  
    series_to_change=series_to_change.map("{:,.2f}%".format)
    return series_to_change;

def format_to_dollars( series_to_change ):  
    series_to_change=series_to_change.map("${:,.0f}".format)
    return series_to_change;

def format_to_plain_number(series_to_change):
    series_to_change=series_to_change.replace({'\%': '','\$': '', ',': ''}, regex=True)
    series_to_change = series_to_change.apply(pd.to_numeric)
    return series_to_change;


```


```python
#Since the threshold for passing was left undeclared in the assignment, this allows the user to enter it in
set_passing_grade = True
while (set_passing_grade):
    passing_grade=int(input("Please enter the lowest, numeric, passing grade: "))
    set_passing_grade=False
    if (passing_grade<0) | (passing_grade>100):
        print("Please enter a number between 0 and 100.")
        set_passing_grade=True
    
```

    Please enter the lowest, numeric, passing grade: 70
    


```python
# **District Summary**

# * Create a high level snapshot (in table form) of the district's key metrics, including:
#   * Total Schools
#   * Total Students
#   * Total Budget
#   * Average Math Score
#   * Average Reading Score
#   * % Passing Math
#   * % Passing Reading
#   * Overall Passing Rate (Average of the above two)
total_schools = len(school_data["name"])
total_students = len(student_data["name"])
total_budget = school_data["budget"].sum()
avg_math = student_data["math_score"].mean()
avg_read = student_data["reading_score"].mean()
math_pass_view = student_data.loc[student_data["math_score"]>=passing_grade,:]
pass_math_perc = (len(math_pass_view)/total_students)*100
read_pass_view = student_data.loc[student_data["reading_score"]>=passing_grade,:]
pass_read_perc = (len(read_pass_view)/total_students)*100
overall_pass_perc = (pass_math_perc + pass_read_perc)/2

district_summary=pd.DataFrame({"Total Schools":[total_schools],"Total Students":[total_students],"Total Budget":[total_budget],"Average Math Score":[avg_math],
                              "Average Reading Score":[avg_read],"% Passing Math":[pass_math_perc],"% Passing Reading":[pass_read_perc],"Overall Passing Rate":[overall_pass_perc]})
district_summary=district_summary[["Total Schools","Total Students","Total Budget","Average Math Score","Average Reading Score","% Passing Math","% Passing Reading","Overall Passing Rate"]]

#format before printing out
to_perc=["% Passing Math","% Passing Reading", "Overall Passing Rate"]
for x in to_perc:
    district_summary[x]=format_to_perc(district_summary[x])

to_plain=["Average Math Score","Average Reading Score"]
for x in to_plain:
    district_summary[x]=format_to_nice_number(district_summary[x])
    
to_dollars=["Total Budget"]
for x in to_dollars:
    district_summary[x]=format_to_dollars(district_summary[x])

district_summary
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39170</td>
      <td>$24,649,428</td>
      <td>79</td>
      <td>82</td>
      <td>74.98%</td>
      <td>85.81%</td>
      <td>80.39%</td>
    </tr>
  </tbody>
</table>
</div>




```python
#rename the school name columns to match so that both dataframes can be merged
school_data=school_data.rename(columns={"name":"school name"})
student_data=student_data.rename(columns={"school":"school name"})

```


```python
# **School Summary**

# * Create an overview table that summarizes key metrics about each school, including:
#   * School Name
#   * School Type
#   * Total Students
#   * Total School Budget
#   * Per Student Budget
#   * Average Math Score
#   * Average Reading Score
#   * % Passing Math
#   * % Passing Reading
#   * Overall Passing Rate (Average of the above two)
all_stats =  pd.merge(school_data, student_data, on="school name")
#all_stats
```


```python
sorted_stats=all_stats.groupby(by="school name")
school_summary=pd.DataFrame(sorted_stats.mean())
school_summary["type"]=sorted_stats["type"].unique()#.astype(str)
school_summary["per student budget"]=school_summary["budget"]/school_summary["size"]

only_pass_math=pd.DataFrame(all_stats.loc[all_stats["math_score"]>=passing_grade,["school name","math_score"]])
sorted_math=only_pass_math.groupby(by="school name")
school_summary["% passing math"]=sorted_math["school name"].count()/school_summary["size"]*100

only_pass_reading=pd.DataFrame(all_stats.loc[all_stats["reading_score"]>=passing_grade,["school name","reading_score"]])
sorted_reading=only_pass_reading.groupby(by="school name")
school_summary["% passing math"]=sorted_math["school name"].count()/school_summary["size"]*100
school_summary["% passing reading"]=sorted_reading["school name"].count()/school_summary["size"]*100
school_summary["overall passing rate"]=(school_summary["% passing math"]+school_summary["% passing reading"])/2

#format

school_summary=school_summary.rename(columns={"per student budget":"Budget Per Student","size":"Total Students","budget":"Budget","reading_score":"Average Reading Score",
                                              "math_score":"Average Math Score","type":"School Type","% passing math":"% Passing Math",
                                              "% passing reading":"% Passing Reading","overall passing rate":"Overall Passing Rate"})
school_summary = school_summary.drop("Student ID",1)
school_summary["Total Students"]=format_to_nice_number(school_summary["Total Students"])
school_summary["Budget"]=format_to_dollars(school_summary["Budget"])
school_summary["Budget Per Student"]=format_to_dollars(school_summary["Budget Per Student"])
school_summary["% Passing Math"]=format_to_perc(school_summary["% Passing Math"])
school_summary["% Passing Reading"]=format_to_perc(school_summary["% Passing Reading"])
school_summary["Overall Passing Rate"]=format_to_perc(school_summary["Overall Passing Rate"])
school_summary
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School ID</th>
      <th>Total Students</th>
      <th>Budget</th>
      <th>Average Reading Score</th>
      <th>Average Math Score</th>
      <th>School Type</th>
      <th>Budget Per Student</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>school name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>7.0</td>
      <td>4,976</td>
      <td>$3,124,928</td>
      <td>81.033963</td>
      <td>77.048432</td>
      <td>[District]</td>
      <td>$628</td>
      <td>66.68%</td>
      <td>81.93%</td>
      <td>74.31%</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>6.0</td>
      <td>1,858</td>
      <td>$1,081,356</td>
      <td>83.975780</td>
      <td>83.061895</td>
      <td>[Charter]</td>
      <td>$582</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.59%</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>1.0</td>
      <td>2,949</td>
      <td>$1,884,411</td>
      <td>81.158020</td>
      <td>76.711767</td>
      <td>[District]</td>
      <td>$639</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>13.0</td>
      <td>2,739</td>
      <td>$1,763,916</td>
      <td>80.746258</td>
      <td>77.102592</td>
      <td>[District]</td>
      <td>$644</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.80%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>4.0</td>
      <td>1,468</td>
      <td>$917,500</td>
      <td>83.816757</td>
      <td>83.351499</td>
      <td>[Charter]</td>
      <td>$625</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>3.0</td>
      <td>4,635</td>
      <td>$3,022,020</td>
      <td>80.934412</td>
      <td>77.289752</td>
      <td>[District]</td>
      <td>$652</td>
      <td>66.75%</td>
      <td>80.86%</td>
      <td>73.81%</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>8.0</td>
      <td>427</td>
      <td>$248,087</td>
      <td>83.814988</td>
      <td>83.803279</td>
      <td>[Charter]</td>
      <td>$581</td>
      <td>92.51%</td>
      <td>96.25%</td>
      <td>94.38%</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>0.0</td>
      <td>2,917</td>
      <td>$1,910,635</td>
      <td>81.182722</td>
      <td>76.629414</td>
      <td>[District]</td>
      <td>$655</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>12.0</td>
      <td>4,761</td>
      <td>$3,094,650</td>
      <td>80.966394</td>
      <td>77.072464</td>
      <td>[District]</td>
      <td>$650</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>9.0</td>
      <td>962</td>
      <td>$585,858</td>
      <td>84.044699</td>
      <td>83.839917</td>
      <td>[Charter]</td>
      <td>$609</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>11.0</td>
      <td>3,999</td>
      <td>$2,547,363</td>
      <td>80.744686</td>
      <td>76.842711</td>
      <td>[District]</td>
      <td>$637</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.29%</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>2.0</td>
      <td>1,761</td>
      <td>$1,056,600</td>
      <td>83.725724</td>
      <td>83.359455</td>
      <td>[Charter]</td>
      <td>$600</td>
      <td>93.87%</td>
      <td>95.85%</td>
      <td>94.86%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>14.0</td>
      <td>1,635</td>
      <td>$1,043,130</td>
      <td>83.848930</td>
      <td>83.418349</td>
      <td>[Charter]</td>
      <td>$638</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>5.0</td>
      <td>2,283</td>
      <td>$1,319,574</td>
      <td>83.989488</td>
      <td>83.274201</td>
      <td>[Charter]</td>
      <td>$578</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.20%</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>10.0</td>
      <td>1,800</td>
      <td>$1,049,400</td>
      <td>83.955000</td>
      <td>83.682222</td>
      <td>[Charter]</td>
      <td>$583</td>
      <td>93.33%</td>
      <td>96.61%</td>
      <td>94.97%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# **Top Performing Schools (By Passing Rate)**

# * Create a table that highlights the top 5 performing schools based on Overall Passing Rate. Include:
#   * School Name
#   * School Type
#   * Total Students
#   * Total School Budget
#   * Per Student Budget
#   * Average Math Score
#   * Average Reading Score
#   * % Passing Math
#   * % Passing Reading
#   * Overall Passing Rate (Average of the above two)

top_schools=school_summary.sort_values(["Overall Passing Rate"], ascending=False)
top_schools=top_schools.iloc[0:5,:]
top_schools=top_schools[["School Type","Total Students","Budget","Budget Per Student","Average Math Score","Average Reading Score","% Passing Math","% Passing Reading","Overall Passing Rate"]]
top_schools

```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Budget</th>
      <th>Budget Per Student</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>school name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>[Charter]</td>
      <td>1,858</td>
      <td>$1,081,356</td>
      <td>$582</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.59%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>[Charter]</td>
      <td>1,635</td>
      <td>$1,043,130</td>
      <td>$638</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>[Charter]</td>
      <td>1,468</td>
      <td>$917,500</td>
      <td>$625</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>[Charter]</td>
      <td>962</td>
      <td>$585,858</td>
      <td>$609</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>[Charter]</td>
      <td>2,283</td>
      <td>$1,319,574</td>
      <td>$578</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.20%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# **Bottom Performing Schools (By Passing Rate)**

bottom_schools=school_summary.sort_values(["Overall Passing Rate"], ascending=True)
bottom_schools=bottom_schools.iloc[0:5,:]
bottom_schools=bottom_schools[["School Type","Total Students","Budget","Budget Per Student","Average Math Score",
                               "Average Reading Score","% Passing Math","% Passing Reading","Overall Passing Rate"]]
bottom_schools

```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>School Type</th>
      <th>Total Students</th>
      <th>Budget</th>
      <th>Budget Per Student</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>school name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Rodriguez High School</th>
      <td>[District]</td>
      <td>3,999</td>
      <td>$2,547,363</td>
      <td>$637</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.29%</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>[District]</td>
      <td>2,949</td>
      <td>$1,884,411</td>
      <td>$639</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>[District]</td>
      <td>2,917</td>
      <td>$1,910,635</td>
      <td>$655</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>[District]</td>
      <td>4,761</td>
      <td>$3,094,650</td>
      <td>$650</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>[District]</td>
      <td>2,739</td>
      <td>$1,763,916</td>
      <td>$644</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.80%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# **Math Scores by Grade**

# * Create a table that lists the average Math Score for students of each grade level (9th, 10th, 11th, 12th) at each school.
#grade_math_stats=all_stats.groupby(["school name","grade"]).mean()
all_stats["grade"] = pd.Categorical(all_stats["grade"], ["9th", "10th", "11th", "12th"])
grade_math_stats=all_stats.groupby(["school name","grade"]).mean()
grade_math_stats=grade_math_stats[["math_score"]]
grade_math_stats=grade_math_stats.rename(columns={"math_score":"Average Math Score"})
grade_math_stats
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Average Math Score</th>
    </tr>
    <tr>
      <th>school name</th>
      <th>grade</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">Bailey High School</th>
      <th>9th</th>
      <td>77.083676</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>76.996772</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>77.515588</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>76.492218</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Cabrera High School</th>
      <th>9th</th>
      <td>83.094697</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>83.154506</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>82.765560</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>83.277487</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Figueroa High School</th>
      <th>9th</th>
      <td>76.403037</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>76.539974</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>76.884344</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>77.151369</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Ford High School</th>
      <th>9th</th>
      <td>77.361345</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>77.672316</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>76.918058</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>76.179963</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Griffin High School</th>
      <th>9th</th>
      <td>82.044010</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>84.229064</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>83.842105</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>83.356164</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Hernandez High School</th>
      <th>9th</th>
      <td>77.438495</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>77.337408</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>77.136029</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>77.186567</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Holden High School</th>
      <th>9th</th>
      <td>83.787402</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>83.429825</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>85.000000</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>82.855422</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Huang High School</th>
      <th>9th</th>
      <td>77.027251</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>75.908735</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>76.446602</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>77.225641</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Johnson High School</th>
      <th>9th</th>
      <td>77.187857</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>76.691117</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>77.491653</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>76.863248</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Pena High School</th>
      <th>9th</th>
      <td>83.625455</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>83.372000</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>84.328125</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>84.121547</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Rodriguez High School</th>
      <th>9th</th>
      <td>76.859966</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>76.612500</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>76.395626</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>77.690748</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Shelton High School</th>
      <th>9th</th>
      <td>83.420755</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>82.917411</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>83.383495</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>83.778976</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Thomas High School</th>
      <th>9th</th>
      <td>83.590022</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>83.087886</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>83.498795</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>83.497041</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Wilson High School</th>
      <th>9th</th>
      <td>83.085578</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>83.724422</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>83.195326</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>83.035794</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Wright High School</th>
      <th>9th</th>
      <td>83.264706</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>84.010288</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>83.836782</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>




```python

# **Reading Scores by Grade**

# * Create a table that lists the average Reading Score for students of each grade level (9th, 10th, 11th, 12th) at each school.
grade_read_stats=all_stats.groupby(["school name","grade"], sort=True).mean()
grade_read_stats=grade_read_stats[["reading_score"]]
grade_read_stats=grade_read_stats.rename(columns={"reading_score":"Average Reading Score"})
grade_read_stats
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Average Reading Score</th>
    </tr>
    <tr>
      <th>school name</th>
      <th>grade</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">Bailey High School</th>
      <th>9th</th>
      <td>81.303155</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>80.907183</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>80.945643</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>80.912451</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Cabrera High School</th>
      <th>9th</th>
      <td>83.676136</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>84.253219</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>83.788382</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>84.287958</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Figueroa High School</th>
      <th>9th</th>
      <td>81.198598</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>81.408912</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>80.640339</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>81.384863</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Ford High School</th>
      <th>9th</th>
      <td>80.632653</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>81.262712</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>80.403642</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>80.662338</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Griffin High School</th>
      <th>9th</th>
      <td>83.369193</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>83.706897</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>84.288089</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>84.013699</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Hernandez High School</th>
      <th>9th</th>
      <td>80.866860</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>80.660147</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>81.396140</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>80.857143</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Holden High School</th>
      <th>9th</th>
      <td>83.677165</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>83.324561</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>83.815534</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>84.698795</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Huang High School</th>
      <th>9th</th>
      <td>81.290284</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>81.512386</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>81.417476</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>80.305983</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Johnson High School</th>
      <th>9th</th>
      <td>81.260714</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>80.773431</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>80.616027</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>81.227564</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Pena High School</th>
      <th>9th</th>
      <td>83.807273</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>83.612000</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>84.335938</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>84.591160</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Rodriguez High School</th>
      <th>9th</th>
      <td>80.993127</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>80.629808</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>80.864811</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>80.376426</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Shelton High School</th>
      <th>9th</th>
      <td>84.122642</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>83.441964</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>84.373786</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>82.781671</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Thomas High School</th>
      <th>9th</th>
      <td>83.728850</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>84.254157</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>83.585542</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>83.831361</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Wilson High School</th>
      <th>9th</th>
      <td>83.939778</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>84.021452</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>83.764608</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>84.317673</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Wright High School</th>
      <th>9th</th>
      <td>83.833333</td>
    </tr>
    <tr>
      <th>10th</th>
      <td>83.812757</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>84.156322</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>




```python
#remove the formatting so we can do further calculations

to_change=["Budget Per Student","Overall Passing Rate","% Passing Math","% Passing Reading"]
for x in to_change:
    school_summary[x]=format_to_plain_number(school_summary[x])

```


```python
# **Scores by School Spending**

# * Create a table that breaks down school performances based on average Spending Ranges (Per Student). Use 4 reasonable bins to group school spending. Include in the table each of the following:
#   * Average Math Score
#   * Average Reading Score
#   * % Passing Math
#   * % Passing Reading
#   * Overall Passing Rate (Average of the above two)

spending_stats=pd.cut(school_summary["Budget Per Student"], [-float("inf"),594,616,638,float("inf")], labels=["1. low funding $0-594", "2. med funding $594-616", "3. high funding $616-638","4. top funding $638+"],include_lowest=True)
spending_stats_df=pd.DataFrame(spending_stats)
spending_scores=school_summary
spending_scores["Spending Level"]=spending_stats_df
spending_scores_df=pd.DataFrame(spending_scores)
spending_summary=spending_scores_df.groupby(["Spending Level","school name"]).mean()

#formatting
spending_summary=spending_summary[["Average Math Score","Average Reading Score","% Passing Math","% Passing Reading","Overall Passing Rate"]]
spending_summary["% Passing Math"]=format_to_perc(spending_summary["% Passing Math"])
spending_summary["% Passing Reading"]=format_to_perc(spending_summary["% Passing Reading"])
spending_summary["Overall Passing Rate"]=format_to_perc(spending_summary["Overall Passing Rate"])
spending_summary
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Level</th>
      <th>school name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">1. low funding $0-594</th>
      <th>Cabrera High School</th>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.59%</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>92.51%</td>
      <td>96.25%</td>
      <td>94.38%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.20%</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>93.33%</td>
      <td>96.61%</td>
      <td>94.97%</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">2. med funding $594-616</th>
      <th>Pena High School</th>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>93.87%</td>
      <td>95.85%</td>
      <td>94.86%</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">3. high funding $616-638</th>
      <th>Bailey High School</th>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>66.68%</td>
      <td>81.93%</td>
      <td>74.31%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.29%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
    <tr>
      <th rowspan="5" valign="top">4. top funding $638+</th>
      <th>Figueroa High School</th>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.80%</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>66.75%</td>
      <td>80.86%</td>
      <td>73.81%</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
  </tbody>
</table>
</div>




```python
#remove formatting for more calculations

to_change=["Overall Passing Rate","% Passing Math","% Passing Reading"]
for x in to_change:
    spending_summary[x]=format_to_plain_number(spending_summary[x])
```


```python
finale_spending=spending_summary.groupby(["Spending Level"]).mean()
finale_spending=finale_spending[["Average Math Score","Average Reading Score","% Passing Math","% Passing Reading","Overall Passing Rate"]]

#format 

for x in to_change:
    finale_spending[x]=format_to_perc(finale_spending[x])

finale_spending
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Spending Level</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1. low funding $0-594</th>
      <td>83.455399</td>
      <td>83.933814</td>
      <td>93.46%</td>
      <td>96.61%</td>
      <td>95.03%</td>
    </tr>
    <tr>
      <th>2. med funding $594-616</th>
      <td>83.599686</td>
      <td>83.885211</td>
      <td>94.23%</td>
      <td>95.90%</td>
      <td>95.06%</td>
    </tr>
    <tr>
      <th>3. high funding $616-638</th>
      <td>80.165248</td>
      <td>82.361084</td>
      <td>79.93%</td>
      <td>89.15%</td>
      <td>84.54%</td>
    </tr>
    <tr>
      <th>4. top funding $638+</th>
      <td>76.961198</td>
      <td>80.997561</td>
      <td>66.56%</td>
      <td>80.69%</td>
      <td>73.62%</td>
    </tr>
  </tbody>
</table>
</div>




```python
#remove formatting

school_summary["Total Students"]=format_to_plain_number(school_summary["Total Students"])
```


```python
# **Scores by School Size**

# * Repeat the above breakdown, but this time group schools based on a reasonable approximation of school size (Small, Medium, Large).

size_stats=pd.cut(school_summary["Total Students"], 3,labels=["Small","Medium","Large"], include_lowest=True)
size_stats_df=pd.DataFrame(size_stats)
size_scores=school_summary
size_scores["size category"]=size_stats_df
size_scores_df=pd.DataFrame(size_scores)
size_summary=size_scores_df.groupby(["size category","school name"]).mean()

size_summary=size_summary[["Average Math Score","Average Reading Score","% Passing Math","% Passing Reading","Overall Passing Rate"]]

for x in to_change:
    size_summary[x]=format_to_perc(size_summary[x])

size_summary
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>size category</th>
      <th>school name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="4" valign="top">Large</th>
      <th>Bailey High School</th>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>66.68%</td>
      <td>81.93%</td>
      <td>74.31%</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>66.75%</td>
      <td>80.86%</td>
      <td>73.81%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.29%</td>
    </tr>
    <tr>
      <th rowspan="4" valign="top">Medium</th>
      <th>Figueroa High School</th>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.80%</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.20%</td>
    </tr>
    <tr>
      <th rowspan="7" valign="top">Small</th>
      <th>Cabrera High School</th>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.59%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>92.51%</td>
      <td>96.25%</td>
      <td>94.38%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>93.87%</td>
      <td>95.85%</td>
      <td>94.86%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>93.33%</td>
      <td>96.61%</td>
      <td>94.97%</td>
    </tr>
  </tbody>
</table>
</div>




```python
#remove formatting
for x in to_change:
    size_summary[x]=format_to_plain_number(size_summary[x])
```


```python
finale_size=size_summary.groupby(["size category"]).mean()
finale_size=finale_size[["Average Math Score","Average Reading Score","% Passing Math","% Passing Reading","Overall Passing Rate"]]
for x in to_change:
    finale_size[x]=format_to_perc(finale_size[x])
finale_size
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>size category</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Large</th>
      <td>77.063340</td>
      <td>80.919864</td>
      <td>66.47%</td>
      <td>81.06%</td>
      <td>73.76%</td>
    </tr>
    <tr>
      <th>Medium</th>
      <td>78.429493</td>
      <td>81.769122</td>
      <td>73.46%</td>
      <td>84.47%</td>
      <td>78.97%</td>
    </tr>
    <tr>
      <th>Small</th>
      <td>83.502373</td>
      <td>83.883125</td>
      <td>93.58%</td>
      <td>96.59%</td>
      <td>95.09%</td>
    </tr>
  </tbody>
</table>
</div>




```python
#remove formatting
for x in to_change:
    finale_size[x]=format_to_plain_number(finale_size[x])
```


```python
# **Scores by School Type**

# * Repeat the above breakdown, but this time group schools based on school type (Charter vs. District).

school_summary["School Type"]=school_summary["School Type"].astype(str)

#finding a subtotal for all the school in the "type" category
#This was an extra step that I do not think was required, but one that I thought was helpful in evaluating the data
temp_summary = pd.DataFrame(school_summary)
finale=pd.DataFrame(temp_summary.groupby(["School Type"]).mean())
finale=finale[["Average Math Score","Average Reading Score","% Passing Math","% Passing Reading","Overall Passing Rate"]]
finale["school name"]="All schools in type category"
finale["School Type"]=finale.index
finale=finale.set_index('school name')


#add the subtotal to the other totals
type_summary = pd.concat([school_summary,finale])

type_summary=type_summary.groupby(["School Type","school name"]).mean()

type_summary=type_summary[["Average Math Score","Average Reading Score","% Passing Math","% Passing Reading","Overall Passing Rate"]]

#format
to_change=["% Passing Math","% Passing Reading","Overall Passing Rate"]
for x in to_change:
    type_summary[x]=format_to_perc(type_summary[x])

type_summary

```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th>school name</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="9" valign="top">['Charter']</th>
      <th>All schools in type category</th>
      <td>83.473852</td>
      <td>83.896421</td>
      <td>93.62%</td>
      <td>96.59%</td>
      <td>95.10%</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>95.59%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>92.51%</td>
      <td>96.25%</td>
      <td>94.38%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>95.27%</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>93.87%</td>
      <td>95.85%</td>
      <td>94.86%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>95.29%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>95.20%</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>93.33%</td>
      <td>96.61%</td>
      <td>94.97%</td>
    </tr>
    <tr>
      <th rowspan="8" valign="top">['District']</th>
      <th>All schools in type category</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>66.55%</td>
      <td>80.80%</td>
      <td>73.67%</td>
    </tr>
    <tr>
      <th>Bailey High School</th>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>66.68%</td>
      <td>81.93%</td>
      <td>74.31%</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>73.36%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>73.80%</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>66.75%</td>
      <td>80.86%</td>
      <td>73.81%</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>73.50%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>73.64%</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>73.29%</td>
    </tr>
  </tbody>
</table>
</div>




```python
#remove format for further calculations
for x in to_change:
    type_summary[x]=format_to_plain_number(type_summary[x])
```


```python
#I was not sure if the "client" wanted the above data, by each school, or this simplified version, so I included both
finale=type_summary.groupby(["School Type"]).mean()
finale=finale[["Average Math Score","Average Reading Score","% Passing Math","% Passing Reading","Overall Passing Rate"]]

#Format

for x in to_change:
    finale[x]=format_to_perc(finale[x])

finale
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>['Charter']</th>
      <td>83.473852</td>
      <td>83.896421</td>
      <td>93.62%</td>
      <td>96.59%</td>
      <td>95.10%</td>
    </tr>
    <tr>
      <th>['District']</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>66.55%</td>
      <td>80.80%</td>
      <td>73.67%</td>
    </tr>
  </tbody>
</table>
</div>

