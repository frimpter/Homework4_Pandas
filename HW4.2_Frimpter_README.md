

```python
# HW4.2 ACADEMY OF PY

import os
import numpy as np
import pandas as pd

# Pull in the files and review variables

schools = "/Users/frimpter/Documents/data_science/ru_datascience_bootcamp/Homework4 Pandas/schools_complete.csv"
students = "/Users/frimpter/Documents/data_science/ru_datascience_bootcamp/Homework4 Pandas/students_complete.csv"

schools_df = pd.DataFrame(pd.read_csv(schools))
students_df = pd.DataFrame(pd.read_csv(students))

schools_df = schools_df.rename(columns={"name":"School", "type":"School Type", "size":"Total Students", "budget":"Total Budget"})
students_df = students_df.rename(columns={"school":"School", "grade":"Grade"})

#schools_df.head()
#students_df.head()
```


```python
# DISTRICT SUMMARY

# Calculate number of students passing math, reading and BOTH
mathpass = students_df.loc[students_df["math_score"] >= 70, :] # Students with passing math scores
readpass = students_df.loc[students_df["reading_score"] >= 70, :] # Students with passing reading scores
totalpass = students_df.loc[(students_df["math_score"] >= 70) & (students_df["reading_score"] >= 70), :] # Students passing both math and reading

# Create a Summary Table/df of district-level information
districts_df = pd.DataFrame({
    "Total Schools": [schools_df["School"].count()],
    "Total Students": [schools_df["Total Students"].sum()],
    "Total Budget": [schools_df["Total Budget"].sum()],
    "Average Math Score": [students_df["math_score"].mean()],
    "Passing Math %": [mathpass["math_score"].count() / students_df["name"].count()],
    "Average Reading Score": [students_df["reading_score"].mean()],
    "Passing Reading %": [readpass["reading_score"].count() / students_df["name"].count()],
    "Overall Passing %": [totalpass["Student ID"].count() / students_df["name"].count()]
})

# Format table
districts_df = districts_df[["Total Schools", "Total Students", "Total Budget", "Average Math Score", "Passing Math %", "Average Reading Score", "Passing Reading %", "Overall Passing %"]]
districts_df["Total Students"] = districts_df["Total Students"].map("{:,}".format)
districts_df["Total Budget"] = districts_df["Total Budget"].map("${:,}".format)
districts_df["Average Math Score"] = districts_df["Average Math Score"].map("{:,.1f}".format)
districts_df["Passing Math %"] = districts_df["Passing Math %"].map("{:.1%}".format)
districts_df["Average Reading Score"] = districts_df["Average Reading Score"].map("{:,.1f}".format)
districts_df["Passing Reading %"] = districts_df["Passing Reading %"].map("{:.1%}".format)
districts_df["Overall Passing %"] = districts_df["Overall Passing %"].map("{:.1%}".format)

districts_df
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
      <th>Passing Math %</th>
      <th>Average Reading Score</th>
      <th>Passing Reading %</th>
      <th>Overall Passing %</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39,170</td>
      <td>$24,649,428</td>
      <td>79.0</td>
      <td>75.0%</td>
      <td>81.9</td>
      <td>85.8%</td>
      <td>65.2%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# SCHOOL SUMMARY
# Create a Summary Table of school-level information

# Prepare the schools data to merge by index/School
schools_df = schools_df.sort_values(["School"])
schools_df.set_index("School", inplace=True)
schools_df = schools_df[["School Type", "Total Students", "Total Budget"]]
#schools_df

# Prepare students data to merge by index/School with mean values
scoresgroup = pd.DataFrame(students_df.groupby(["School"]).mean())

# New DataFrame
combo_df = pd.merge(schools_df, scoresgroup, right_index=True, left_index=True)
#combo_df
```


```python
# Format Columns
combo_df = combo_df.rename(columns={
    "reading_score":"Average Reading Score",
    "math_score":"Average Math Score"
})

# Append computed parameters
# Budget Per Student
combo_df[combo_df.columns] = combo_df[combo_df.columns].apply(pd.to_numeric, errors="ignore")
combo_df["Budget Per Student"] = combo_df["Total Budget"] / combo_df["Total Students"]

# Average Math, Reading, Overall Passing scores per school
MathPass = mathpass.groupby(["School"])
combo_df["% Passing Math"] = (MathPass["Student ID"].count() / combo_df["Total Students"])*100
ReadPass = readpass.groupby(["School"])
combo_df["% Passing Reading"] = (ReadPass["Student ID"].count() / combo_df["Total Students"])*100
TotalPass = totalpass.groupby(["School"])
combo_df["% Passing Overall"] = (TotalPass["Student ID"].count() / combo_df["Total Students"])*100

# Arrange table columns
combo_df = combo_df[["School Type", "Total Students", "Total Budget", "Budget Per Student", "Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]]

# Format values
combo_df["Total Students"] = combo_df["Total Students"].map("{:,}".format)
combo_df["Total Budget"] = combo_df["Total Budget"].map("${:,}".format)
combo_df["Budget Per Student"] = combo_df["Budget Per Student"].map("${:,.0f}".format)
combo_df["% Passing Math"] = combo_df["% Passing Math"].map("{:.1f}%".format)
combo_df["% Passing Reading"] = combo_df["% Passing Reading"].map("{:.1f}%".format)
combo_df["% Passing Overall"] = combo_df["% Passing Overall"].map("{:.1f}%".format)

round(combo_df, 1)
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
      <th>Total Budget</th>
      <th>Budget Per Student</th>
      <th>Average Math Score</th>
      <th>% Passing Math</th>
      <th>Average Reading Score</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>School</th>
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
      <td>District</td>
      <td>4,976</td>
      <td>$3,124,928</td>
      <td>$628</td>
      <td>77.0</td>
      <td>66.7%</td>
      <td>81.0</td>
      <td>81.9%</td>
      <td>54.6%</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>Charter</td>
      <td>1,858</td>
      <td>$1,081,356</td>
      <td>$582</td>
      <td>83.1</td>
      <td>94.1%</td>
      <td>84.0</td>
      <td>97.0%</td>
      <td>91.3%</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411</td>
      <td>$639</td>
      <td>76.7</td>
      <td>66.0%</td>
      <td>81.2</td>
      <td>80.7%</td>
      <td>53.2%</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>District</td>
      <td>2,739</td>
      <td>$1,763,916</td>
      <td>$644</td>
      <td>77.1</td>
      <td>68.3%</td>
      <td>80.7</td>
      <td>79.3%</td>
      <td>54.3%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1,468</td>
      <td>$917,500</td>
      <td>$625</td>
      <td>83.4</td>
      <td>93.4%</td>
      <td>83.8</td>
      <td>97.1%</td>
      <td>90.6%</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4,635</td>
      <td>$3,022,020</td>
      <td>$652</td>
      <td>77.3</td>
      <td>66.8%</td>
      <td>80.9</td>
      <td>80.9%</td>
      <td>53.5%</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>Charter</td>
      <td>427</td>
      <td>$248,087</td>
      <td>$581</td>
      <td>83.8</td>
      <td>92.5%</td>
      <td>83.8</td>
      <td>96.3%</td>
      <td>89.2%</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2,917</td>
      <td>$1,910,635</td>
      <td>$655</td>
      <td>76.6</td>
      <td>65.7%</td>
      <td>81.2</td>
      <td>81.3%</td>
      <td>53.5%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4,761</td>
      <td>$3,094,650</td>
      <td>$650</td>
      <td>77.1</td>
      <td>66.1%</td>
      <td>81.0</td>
      <td>81.2%</td>
      <td>53.5%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858</td>
      <td>$609</td>
      <td>83.8</td>
      <td>94.6%</td>
      <td>84.0</td>
      <td>95.9%</td>
      <td>90.5%</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3,999</td>
      <td>$2,547,363</td>
      <td>$637</td>
      <td>76.8</td>
      <td>66.4%</td>
      <td>80.7</td>
      <td>80.2%</td>
      <td>53.0%</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>Charter</td>
      <td>1,761</td>
      <td>$1,056,600</td>
      <td>$600</td>
      <td>83.4</td>
      <td>93.9%</td>
      <td>83.7</td>
      <td>95.9%</td>
      <td>89.9%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1,635</td>
      <td>$1,043,130</td>
      <td>$638</td>
      <td>83.4</td>
      <td>93.3%</td>
      <td>83.8</td>
      <td>97.3%</td>
      <td>90.9%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2,283</td>
      <td>$1,319,574</td>
      <td>$578</td>
      <td>83.3</td>
      <td>93.9%</td>
      <td>84.0</td>
      <td>96.5%</td>
      <td>90.6%</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>Charter</td>
      <td>1,800</td>
      <td>$1,049,400</td>
      <td>$583</td>
      <td>83.7</td>
      <td>93.3%</td>
      <td>84.0</td>
      <td>96.6%</td>
      <td>90.3%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# TOP 5 PERFORMERS
top_five = combo_df.sort_values("% Passing Overall", ascending=False)
top_five.head()
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
      <th>Total Budget</th>
      <th>Budget Per Student</th>
      <th>Average Math Score</th>
      <th>% Passing Math</th>
      <th>Average Reading Score</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>School</th>
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
      <td>Charter</td>
      <td>1,858</td>
      <td>$1,081,356</td>
      <td>$582</td>
      <td>83.061895</td>
      <td>94.1%</td>
      <td>83.975780</td>
      <td>97.0%</td>
      <td>91.3%</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>Charter</td>
      <td>1,635</td>
      <td>$1,043,130</td>
      <td>$638</td>
      <td>83.418349</td>
      <td>93.3%</td>
      <td>83.848930</td>
      <td>97.3%</td>
      <td>90.9%</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>Charter</td>
      <td>1,468</td>
      <td>$917,500</td>
      <td>$625</td>
      <td>83.351499</td>
      <td>93.4%</td>
      <td>83.816757</td>
      <td>97.1%</td>
      <td>90.6%</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>Charter</td>
      <td>2,283</td>
      <td>$1,319,574</td>
      <td>$578</td>
      <td>83.274201</td>
      <td>93.9%</td>
      <td>83.989488</td>
      <td>96.5%</td>
      <td>90.6%</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>Charter</td>
      <td>962</td>
      <td>$585,858</td>
      <td>$609</td>
      <td>83.839917</td>
      <td>94.6%</td>
      <td>84.044699</td>
      <td>95.9%</td>
      <td>90.5%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# WORST 5 PERFORMERS
top_five.tail()
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
      <th>Total Budget</th>
      <th>Budget Per Student</th>
      <th>Average Math Score</th>
      <th>% Passing Math</th>
      <th>Average Reading Score</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>School</th>
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
      <th>Hernandez High School</th>
      <td>District</td>
      <td>4,635</td>
      <td>$3,022,020</td>
      <td>$652</td>
      <td>77.289752</td>
      <td>66.8%</td>
      <td>80.934412</td>
      <td>80.9%</td>
      <td>53.5%</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>District</td>
      <td>2,917</td>
      <td>$1,910,635</td>
      <td>$655</td>
      <td>76.629414</td>
      <td>65.7%</td>
      <td>81.182722</td>
      <td>81.3%</td>
      <td>53.5%</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>District</td>
      <td>4,761</td>
      <td>$3,094,650</td>
      <td>$650</td>
      <td>77.072464</td>
      <td>66.1%</td>
      <td>80.966394</td>
      <td>81.2%</td>
      <td>53.5%</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>District</td>
      <td>2,949</td>
      <td>$1,884,411</td>
      <td>$639</td>
      <td>76.711767</td>
      <td>66.0%</td>
      <td>81.158020</td>
      <td>80.7%</td>
      <td>53.2%</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>District</td>
      <td>3,999</td>
      <td>$2,547,363</td>
      <td>$637</td>
      <td>76.842711</td>
      <td>66.4%</td>
      <td>80.744686</td>
      <td>80.2%</td>
      <td>53.0%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# MATH SCORES BY GRADE AND SCHOOL
SchoolGrade = pd.DataFrame(students_df.groupby(["School", "Grade"]).mean().unstack())
SchoolGrade = SchoolGrade.rename(columns={"math_score":"Average Math Score", "reading_score":"Average Reading Score"})
MathGrade = SchoolGrade[["Average Math Score"]]

round(MathGrade, 1)
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
    <tr>
      <th></th>
      <th colspan="4" halign="left">Average Math Score</th>
    </tr>
    <tr>
      <th>Grade</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
      <th>9th</th>
    </tr>
    <tr>
      <th>School</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.0</td>
      <td>77.5</td>
      <td>76.5</td>
      <td>77.1</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.2</td>
      <td>82.8</td>
      <td>83.3</td>
      <td>83.1</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.5</td>
      <td>76.9</td>
      <td>77.2</td>
      <td>76.4</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.7</td>
      <td>76.9</td>
      <td>76.2</td>
      <td>77.4</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>84.2</td>
      <td>83.8</td>
      <td>83.4</td>
      <td>82.0</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.3</td>
      <td>77.1</td>
      <td>77.2</td>
      <td>77.4</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.4</td>
      <td>85.0</td>
      <td>82.9</td>
      <td>83.8</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>75.9</td>
      <td>76.4</td>
      <td>77.2</td>
      <td>77.0</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>76.7</td>
      <td>77.5</td>
      <td>76.9</td>
      <td>77.2</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.4</td>
      <td>84.3</td>
      <td>84.1</td>
      <td>83.6</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.6</td>
      <td>76.4</td>
      <td>77.7</td>
      <td>76.9</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>82.9</td>
      <td>83.4</td>
      <td>83.8</td>
      <td>83.4</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.1</td>
      <td>83.5</td>
      <td>83.5</td>
      <td>83.6</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.7</td>
      <td>83.2</td>
      <td>83.0</td>
      <td>83.1</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>84.0</td>
      <td>83.8</td>
      <td>83.6</td>
      <td>83.3</td>
    </tr>
  </tbody>
</table>
</div>




```python
# READING SCORES BY GRADE AND SCHOOL
ReadGrade = SchoolGrade[["Average Reading Score"]]

round(ReadGrade, 1)
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
    <tr>
      <th></th>
      <th colspan="4" halign="left">Average Reading Score</th>
    </tr>
    <tr>
      <th>Grade</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
      <th>9th</th>
    </tr>
    <tr>
      <th>School</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>80.9</td>
      <td>80.9</td>
      <td>80.9</td>
      <td>81.3</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>84.3</td>
      <td>83.8</td>
      <td>84.3</td>
      <td>83.7</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.4</td>
      <td>80.6</td>
      <td>81.4</td>
      <td>81.2</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>81.3</td>
      <td>80.4</td>
      <td>80.7</td>
      <td>80.6</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.7</td>
      <td>84.3</td>
      <td>84.0</td>
      <td>83.4</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.7</td>
      <td>81.4</td>
      <td>80.9</td>
      <td>80.9</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.3</td>
      <td>83.8</td>
      <td>84.7</td>
      <td>83.7</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.5</td>
      <td>81.4</td>
      <td>80.3</td>
      <td>81.3</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>80.8</td>
      <td>80.6</td>
      <td>81.2</td>
      <td>81.3</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.6</td>
      <td>84.3</td>
      <td>84.6</td>
      <td>83.8</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.6</td>
      <td>80.9</td>
      <td>80.4</td>
      <td>81.0</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.4</td>
      <td>84.4</td>
      <td>82.8</td>
      <td>84.1</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>84.3</td>
      <td>83.6</td>
      <td>83.8</td>
      <td>83.7</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>84.0</td>
      <td>83.8</td>
      <td>84.3</td>
      <td>83.9</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.8</td>
      <td>84.2</td>
      <td>84.1</td>
      <td>83.8</td>
    </tr>
  </tbody>
</table>
</div>




```python
# SCORES BY SPENDING PER STUDENT

# Remove combo_df formatting and convert all to numeric (leave non-numeric values, as in 'District') 
combo_df = combo_df.replace({"\$":"", "%":"", ",":""}, regex=True)
combo_df[combo_df.columns] = combo_df[combo_df.columns].apply(pd.to_numeric, errors="ignore")

# Add spending category
bins = [550, 600, 650, 700]
labels = ["\$550 to \$600", "\$601 to \$650", "\$651 to \$700"]
combo_df["Spending Category"] = pd.cut(combo_df["Budget Per Student"], bins, labels=labels)

combo_df = combo_df[["Spending Category", "Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]]

#combo_df
```


```python
# PRESENT SCORES BY SPENDING PER STUDENT CATEGORY

spending_pivot = pd.DataFrame(combo_df.pivot_table(index="Spending Category", values=["Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]))
spending_pivot = spending_pivot[["Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]]

# Format values
spending_pivot["% Passing Math"] = spending_pivot["% Passing Math"].map("{:.1f}%".format)
spending_pivot["% Passing Reading"] = spending_pivot["% Passing Reading"].map("{:.1f}%".format)
spending_pivot["% Passing Overall"] = spending_pivot["% Passing Overall"].map("{:.1f}%".format)

round(spending_pivot, 1)
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
      <th>% Passing Math</th>
      <th>Average Reading Score</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>Spending Category</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>\$550 to \$600</th>
      <td>83.4</td>
      <td>93.5%</td>
      <td>83.9</td>
      <td>96.5%</td>
      <td>90.3%</td>
    </tr>
    <tr>
      <th>\$601 to \$650</th>
      <td>79.4</td>
      <td>76.8%</td>
      <td>82.0</td>
      <td>86.7%</td>
      <td>67.6%</td>
    </tr>
    <tr>
      <th>\$651 to \$700</th>
      <td>77.0</td>
      <td>66.2%</td>
      <td>81.1</td>
      <td>81.1%</td>
      <td>53.5%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# SCORES BY SCHOOL SIZE

# Add size categories
bins = [0, 999, 1999, 2999, 3999, 5000]
labels = ["<1,000", "1,000 to 1,999", "2,000 to 2,999", "3,000 to 3,999", "4,000 to 5,000"]
combo_df["Student Population"] = pd.cut(schools_df["Total Students"], bins, labels=labels)

combo_df = combo_df[["Student Population", "Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]]

#combo_df
```


```python
# PRESENT SCORES BY SCHOOL SIZE GROUPING

size_pivot = pd.DataFrame(combo_df.pivot_table(index="Student Population", values=["Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]))
size_pivot = size_pivot[["Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]]

# Format values
size_pivot["% Passing Math"] = size_pivot["% Passing Math"].map("{:.1f}%".format)
size_pivot["% Passing Reading"] = size_pivot["% Passing Reading"].map("{:.1f}%".format)
size_pivot["% Passing Overall"] = size_pivot["% Passing Overall"].map("{:.1f}%".format)

round(size_pivot, 1)
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
      <th>% Passing Math</th>
      <th>Average Reading Score</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
    </tr>
    <tr>
      <th>Student Population</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt;1,000</th>
      <td>83.8</td>
      <td>93.5%</td>
      <td>83.9</td>
      <td>96.1%</td>
      <td>89.8%</td>
    </tr>
    <tr>
      <th>1,000 to 1,999</th>
      <td>83.4</td>
      <td>93.6%</td>
      <td>83.9</td>
      <td>96.8%</td>
      <td>90.6%</td>
    </tr>
    <tr>
      <th>2,000 to 2,999</th>
      <td>78.4</td>
      <td>73.5%</td>
      <td>81.8</td>
      <td>84.5%</td>
      <td>62.9%</td>
    </tr>
    <tr>
      <th>3,000 to 3,999</th>
      <td>76.8</td>
      <td>66.4%</td>
      <td>80.7</td>
      <td>80.2%</td>
      <td>53.0%</td>
    </tr>
    <tr>
      <th>4,000 to 5,000</th>
      <td>77.1</td>
      <td>66.5%</td>
      <td>81.0</td>
      <td>81.3%</td>
      <td>53.9%</td>
    </tr>
  </tbody>
</table>
</div>




```python
# PRESENT SCORES BY SCHOOL TYPE

combo_df["School Type"] = schools_df["School Type"] # Add School Type into combo_df

type_pivot = pd.DataFrame(combo_df.pivot_table(index="School Type", values=["Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]))
type_pivot = type_pivot[["Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]]

# Format values
type_pivot["% Passing Math"] = type_pivot["% Passing Math"].map("{:.1f}%".format)
type_pivot["% Passing Reading"] = type_pivot["% Passing Reading"].map("{:.1f}%".format)
type_pivot["% Passing Overall"] = type_pivot["% Passing Overall"].map("{:.1f}%".format)

round(type_pivot, 1)
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
      <th>% Passing Math</th>
      <th>Average Reading Score</th>
      <th>% Passing Reading</th>
      <th>% Passing Overall</th>
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
      <th>Charter</th>
      <td>83.5</td>
      <td>93.6%</td>
      <td>83.9</td>
      <td>96.6%</td>
      <td>90.4%</td>
    </tr>
    <tr>
      <th>District</th>
      <td>77.0</td>
      <td>66.6%</td>
      <td>81.0</td>
      <td>80.8%</td>
      <td>53.7%</td>
    </tr>
  </tbody>
</table>
</div>




```python
print("Three Trends from School Data Analysis: ")
print("\n1. In the district, performance is better in Reading than in Math, however little more than half (65.2%) of students pass both reading and math.")
print("\n2. A greater proportion of students at Charter schools pass math, reading and both compared to District schools.")
print("\n3. These findings suggest that students at smaller schools achieve better performance in all categories, and spending is not necessarily indicative of performance.")

```

    Three Trends from School Data Analysis: 
    
    1. In the district, performance is better in Reading than in Math, however little more than half (65.2%) of students pass both reading and math.
    
    2. A greater proportion of students at Charter schools pass math, reading and both compared to District schools.
    
    3. These findings suggest that students at smaller schools achieve better performance in all categories, and spending is not necessarily indicative of performance.

