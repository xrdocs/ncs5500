---
published: true
date: '2021-03-18 21:23 -0700'
title: NCS-5500 Interface and QoS 1D Scale
author: Vincent Ng
excerpt: NCS-5500 Interface and QoS 1D Scale
position: hidden
---
{% include toc %}

## A New Post

| QSFP56-DD       | QSFP28-DD    | QSFP56          | QSFP28         | QSFP+          |
|:---------------:|:------------:|:---------------:|:--------------:|:--------------:|
| 400G            | 200G         | 200G            | 100G           | 40G            |
|-----------------+--------------+-----------------+----------------|:---------------|
| 2x200G          | 2x100G       | 2x100G          | 4x25G          | 4x10G          |
|-----------------+--------------+-----------------+----------------|:---------------|
| 4x100G          | 8x25G        | 4x50G           |                |                |
|-----------------+--------------+-----------------+----------------|:---------------|
| 8x5 0G          |              |                 |                |                |


## Table Styling in Markdown

<style>
.heatMap {
    width: 70%;
    text-align: center;
}
.heatMap th {
background: grey;
word-wrap: break-word;
text-align: center;
}
.heatMap tr:nth-child(1) { background: red; }
.heatMap tr:nth-child(2) { background: orange; }
.heatMap tr:nth-child(3) { background: green; }
</style>

<div class="heatMap" markdown="1">

| Everything | in this table | is Centered |  and the table will only take up 70% of the screen width  | 
| -- | -- | -- | -- |
| This | is | a | Red Row |
| This | is | an | Orange Row |
| This | is | a | Green Row |

</div>

### New paragraph

<table>
<tr>
    <td rowspan="7">  Combine multiple rows into one column:<br/>
                 Use rowspan = "n" <br/>
                 Merge across n rows<br/>
        </td>
    <td>File identification:</td>
    <td>content</td>
</tr>
<tr>
    <td>first row:</td>
    <td>What should I write?</td>
</tr>
<tr>
    <td>second line:</td>
    <td>Just write it!</td>
</tr>
<tr>
    <td>The third row:</td>
    <td>OK!</td>
</tr>
</table>




