

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
stugroup = students_df.groupby(["School"])
scoresgroup = pd.DataFrame(stugroup.mean())

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
combo_df["Total Students"] = pd.to_numeric(combo_df["Total Students"])
combo_df["Total Budget"] = pd.to_numeric(combo_df["Total Budget"])
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
combo_df["Average Math Score"] = combo_df["Average Math Score"].map("{:.1f}".format)
combo_df["% Passing Math"] = combo_df["% Passing Math"].map("{:.1f}%".format)
combo_df["Average Reading Score"] = combo_df["Average Reading Score"].map("{:.1f}".format)
combo_df["% Passing Reading"] = combo_df["% Passing Reading"].map("{:.1f}%".format)
combo_df["% Passing Overall"] = combo_df["% Passing Overall"].map("{:.1f}%".format)

combo_df
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
      <td>83.1</td>
      <td>94.1%</td>
      <td>84.0</td>
      <td>97.0%</td>
      <td>91.3%</td>
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
      <td>77.3</td>
      <td>66.8%</td>
      <td>80.9</td>
      <td>80.9%</td>
      <td>53.5%</td>
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
  </tbody>
</table>
</div>




```python
# MATH SCORES BY GRADE AND SCHOOL
schoolgrade = students_df.groupby(["School", "Grade"]).mean().unstack()
SchoolGrade = pd.DataFrame(schoolgrade)
SchoolGrade = SchoolGrade.rename(columns={"math_score":"Average Math Score", "reading_score":"Average Reading Score"})
MathGrade = SchoolGrade[["Average Math Score"]]

MathGrade
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
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
      <td>77.083676</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
      <td>83.094697</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
      <td>76.403037</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
      <td>77.361345</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
      <td>82.044010</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
      <td>77.438495</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
      <td>83.787402</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
      <td>77.027251</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
      <td>77.187857</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
      <td>83.625455</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
      <td>76.859966</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
      <td>83.420755</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
      <td>83.590022</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
      <td>83.085578</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
      <td>83.264706</td>
    </tr>
  </tbody>
</table>
</div>




```python
# READING SCORES BY GRADE AND SCHOOL
ReadGrade = SchoolGrade[["Average Reading Score"]]

ReadGrade
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
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
      <td>81.303155</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
      <td>83.676136</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
      <td>81.198598</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
      <td>80.632653</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
      <td>83.369193</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
      <td>80.866860</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
      <td>83.677165</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
      <td>81.290284</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
      <td>81.260714</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
      <td>83.807273</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
      <td>80.993127</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
      <td>84.122642</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
      <td>83.728850</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
      <td>83.939778</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
      <td>83.833333</td>
    </tr>
  </tbody>
</table>
</div>




```python
# SCORES BY SPENDING PER STUDENT

# Make a New (Unformatted) DataFrame similar to combo_df
newcombo_df = pd.merge(schools_df, scoresgroup, right_index=True, left_index=True)

# Format Columns
newcombo_df = newcombo_df.rename(columns={
    "reading_score":"Average Reading Score",
    "math_score":"Average Math Score"
})

# Spending Per Student
newcombo_df["Total Students"] = pd.to_numeric(newcombo_df["Total Students"])
newcombo_df["Total Budget"] = pd.to_numeric(newcombo_df["Total Budget"])
newcombo_df["Spending Per Student"] = newcombo_df["Total Budget"] / newcombo_df["Total Students"]

# Average Math, Reading, Overall Passing scores per school
MathPass = mathpass.groupby(["School"])
newcombo_df["% Passing Math"] = (MathPass["Student ID"].count() / newcombo_df["Total Students"])*100
ReadPass = readpass.groupby(["School"])
newcombo_df["% Passing Reading"] = (ReadPass["Student ID"].count() / newcombo_df["Total Students"])*100
TotalPass = totalpass.groupby(["School"])
newcombo_df["% Passing Overall"] = (TotalPass["Student ID"].count() / newcombo_df["Total Students"])*100

# Check range of average spending values to inform bins
# print(combo_df["Budget Per Student"].min())
# print(combo_df["Budget Per Student"].max())

# Add spending category
bins = [550, 600, 650, 700]
labels = ["\$550 to \$600", "\$601 to \$650", "\$651 to \$700"]
newcombo_df["Spending Category"] = pd.cut(newcombo_df["Spending Per Student"], bins, labels=labels)

newcombo_df = newcombo_df[["Spending Category", "Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]]

#newcombo_df
```


```python
# PRESENT SCORES BY SPENDING PER STUDENT CATEGORY

spending_pivot = pd.DataFrame(newcombo_df.pivot_table(index="Spending Category", values=["Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]))
spending_pivot = spending_pivot[["Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]]

# Format values
spending_pivot["Average Math Score"] = spending_pivot["Average Math Score"].map("{:.1f}".format)
spending_pivot["% Passing Math"] = spending_pivot["% Passing Math"].map("{:.1f}%".format)
spending_pivot["Average Reading Score"] = spending_pivot["Average Reading Score"].map("{:.1f}".format)
spending_pivot["% Passing Reading"] = spending_pivot["% Passing Reading"].map("{:.1f}%".format)
spending_pivot["% Passing Overall"] = spending_pivot["% Passing Overall"].map("{:.1f}%".format)

spending_pivot
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

# Reset newcombo_df DataFrame
newcombo_df = pd.merge(schools_df, scoresgroup, right_index=True, left_index=True)

# Format Columns
newcombo_df = newcombo_df.rename(columns={
    "reading_score":"Average Reading Score",
    "math_score":"Average Math Score"
})

newcombo_df["Total Students"] = pd.to_numeric(newcombo_df["Total Students"])

# Average Math, Reading, Overall Passing scores per school
MathPass = mathpass.groupby(["School"])
newcombo_df["% Passing Math"] = (MathPass["Student ID"].count() / newcombo_df["Total Students"])*100
ReadPass = readpass.groupby(["School"])
newcombo_df["% Passing Reading"] = (ReadPass["Student ID"].count() / newcombo_df["Total Students"])*100
TotalPass = totalpass.groupby(["School"])
newcombo_df["% Passing Overall"] = (TotalPass["Student ID"].count() / newcombo_df["Total Students"])*100

# Check range of average spending values to inform bins
#print(newcombo_df["Total Students"].min())
#print(newcombo_df["Total Students"].max())

# Add size categories
bins = [0, 999, 1999, 2999, 3999, 5000]
labels = ["<1,000", "1,000 to 1,999", "2,000 to 2,999", "3,000 to 3,999", "4,000 to 5,000"]
newcombo_df["Student Population"] = pd.cut(newcombo_df["Total Students"], bins, labels=labels)

newcombo_df = newcombo_df[["Student Population", "Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]]

#newcombo_df
```


```python
# PRESENT SCORES BY SCHOOL SIZE GROUPING

size_pivot = pd.DataFrame(newcombo_df.pivot_table(index="Student Population", values=["Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]))
size_pivot = size_pivot[["Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]]

# Format values
size_pivot["Average Math Score"] = size_pivot["Average Math Score"].map("{:.1f}".format)
size_pivot["% Passing Math"] = size_pivot["% Passing Math"].map("{:.1f}%".format)
size_pivot["Average Reading Score"] = size_pivot["Average Reading Score"].map("{:.1f}".format)
size_pivot["% Passing Reading"] = size_pivot["% Passing Reading"].map("{:.1f}%".format)
size_pivot["% Passing Overall"] = size_pivot["% Passing Overall"].map("{:.1f}%".format)

size_pivot
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
      <td>93.6%</td>
      <td>83.9</td>
      <td>96.1%</td>
      <td>89.9%</td>
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
# SCORES BY SCHOOL TYPE

# Reset newcombo_df DataFrame
newcombo_df = pd.merge(schools_df, scoresgroup, right_index=True, left_index=True)

# Format Columns
newcombo_df = newcombo_df.rename(columns={
    "reading_score":"Average Reading Score",
    "math_score":"Average Math Score"
})

newcombo_df["Total Students"] = pd.to_numeric(newcombo_df["Total Students"])

# Average Math, Reading, Overall Passing scores per school
MathPass = mathpass.groupby(["School"])
newcombo_df["% Passing Math"] = (MathPass["Student ID"].count() / newcombo_df["Total Students"])*100
ReadPass = readpass.groupby(["School"])
newcombo_df["% Passing Reading"] = (ReadPass["Student ID"].count() / newcombo_df["Total Students"])*100
TotalPass = totalpass.groupby(["School"])
newcombo_df["% Passing Overall"] = (TotalPass["Student ID"].count() / newcombo_df["Total Students"])*100

newcombo_df = newcombo_df[["School Type", "Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]]

#newcombo_df
```


```python
# PRESENT SCORES BY SCHOOL TYPE

type_pivot = pd.DataFrame(newcombo_df.pivot_table(index="School Type", values=["Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]))
type_pivot = type_pivot[["Average Math Score", "% Passing Math", "Average Reading Score", "% Passing Reading", "% Passing Overall"]]

# Format values
type_pivot["Average Math Score"] = type_pivot["Average Math Score"].map("{:.1f}".format)
type_pivot["% Passing Math"] = type_pivot["% Passing Math"].map("{:.1f}%".format)
type_pivot["Average Reading Score"] = type_pivot["Average Reading Score"].map("{:.1f}".format)
type_pivot["% Passing Reading"] = type_pivot["% Passing Reading"].map("{:.1f}%".format)
type_pivot["% Passing Overall"] = type_pivot["% Passing Overall"].map("{:.1f}%".format)

type_pivot
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
      <td>66.5%</td>
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

