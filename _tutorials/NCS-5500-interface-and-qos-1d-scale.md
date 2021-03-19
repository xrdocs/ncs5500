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


## Table with only the class specified {#table-with-only-the-class-specified}

<div class="ox-hugo-table my-table" markdown="1">
<div></div>

| h1  | h2  | h3  |
|-----|-----|-----|
| abc | def | ghi |

</div>

Above table is wrapped in a `my-table` class. Just doing that won't
bring any presentation changes to the table.. you'd need to add some
CSS that customizes `.my-table table`.


## Table with only a CSS attribute specified {#table-with-only-a-css-attribute-specified}

<style>.table-nocaption table { width: 50%;  }</style>

<div class="ox-hugo-table table-nocaption" markdown="1">
<div></div>

| h1  | h2  | h3  |
|-----|-----|-----|
| abc | def | ghi |

</div>

Above table get wrapped in the auto-generated class
`table-nocaption`. The specified CSS attribute is auto-set for
`.table-nocaption table`.


## Table with both class and CSS attribute specified {#table-with-both-class-and-css-attribute-specified}

<style>.my-table-2 table { width: 80%;  }</style>

<div class="ox-hugo-table my-table-2" markdown="1">
<div></div>

| h1  | h2  | h3  |
|-----|-----|-----|
| abc | def | ghi |

</div>

Above table get wrapped in the specified class `my-table-2`. The
specified CSS attribute is auto-set for `.my-table-2 table`.


## Table with caption, and CSS attributes specified {#table-with-caption-and-css-attributes-specified}

<style>.table-1 table { text-align: left;  }</style>

<div class="ox-hugo-table table-1" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 1</span>:
  Table with caption, with left-aligned text
</div>

| h1       | h2       | h3       |
|----------|----------|----------|
| abcdefgh | ijklmnop | qrstuvwx |

</div>

Above table get wrapped in the auto-generated class `table-1` (_"1",
because this is the first table with a caption on this page._). The
specified CSS attribute is auto-set for `.table-1 table`.

<style>.table-2 table { text-align: right;  }</style>

<div class="ox-hugo-table table-2" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 2</span>:
  Table with caption, with right-aligned text
</div>

| h1       | h2       | h3       |
|----------|----------|----------|
| abcdefgh | ijklmnop | qrstuvwx |

</div>

<style>.table-3 table { text-align: center;  }</style>

<div class="ox-hugo-table table-3" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 3</span>:
  Table with caption, with center-aligned text
</div>

| h1       | h2       | h3       |
|----------|----------|----------|
| abcdefgh | ijklmnop | qrstuvwx |

</div>


## Table with caption, and both class and CSS attributes specified {#table-with-caption-and-both-class-and-css-attributes-specified}

<style>.my-red-bordered-table table { border: 2px solid red;  }</style>

<div class="ox-hugo-table my-red-bordered-table" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 4</span>:
  Table with caption, class and CSS attributes specified
</div>

| h1  | h2  | h3  |
|-----|-----|-----|
| abc | def | ghi |

</div>

Above table get wrapped in the specified class
`my-red-bordered-table`. The specified CSS attribute is auto-set for
`.my-red-bordered-table table`.

Here's another table with the exact same class as that of the above
table. So the CSS properties do not need to be specified again.

Below table will also show up with a red border.

<div class="ox-hugo-table my-red-bordered-table" markdown="1">
<div></div>

| h1  | h2  | h3  |
|-----|-----|-----|
| jkl | kmo | pqr |

</div>


## Table with verbatim CSS {#table-with-verbatim-css}

There could be times when the CSS couldn't be simply specified using
the `#+attr_css` attribute. In those cases, simply set the table class
using `#+attr_html`, and put the CSS in the `#+begin_export html`
block.

<style>
.tab4 th,
.tab4 td {
    padding: 20px;
    text-align: left;
}
</style>

<div class="ox-hugo-table tab4" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 5</span>:
  Table with verbatim CSS
</div>

| h1  | h2  | h3  |
|-----|-----|-----|
| abc | def | ghi |

</div>


## More table styling examples {#more-table-styling-examples}

[Source: _css-tricks.com_](https://css-tricks.com/complete-guide-table-element/)


### Basic styling {#basic-styling}


#### Uncollapsed borders {#uncollapsed-borders}

<style>
.basic-styling td,
.basic-styling th {
  border: 1px solid #999;
  padding: 0.5rem;
}
</style>

<div class="ox-hugo-table basic-styling" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 6</span>:
  Table with uncollapsed borders
</div>

| Name | ID    | Favorite Color |
|------|-------|----------------|
| Jim  | 00001 | Blue           |
| Sue  | 00002 | Red            |
| Barb | 00003 | Green          |

</div>


#### Collapsed borders {#collapsed-borders}

<style>
.collapsed table {
  border-collapse: collapse;
}
</style>

<div class="ox-hugo-table collapsed basic-styling" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 7</span>:
  Table with collapsed borders
</div>

| Name | ID    | Favorite Color |
|------|-------|----------------|
| Jim  | 00001 | Blue           |
| Sue  | 00002 | Red            |
| Barb | 00003 | Green          |

</div>


### Two Axis Tables {#two-axis-tables}

<style>
.two-axis-table td,
.two-axis-table th {
  width: 4rem;
  height: 2rem;
  border: 1px solid #ccc;
  text-align: center;
}
.two-axis-table th,
.two-axis-table td:nth-child(1) {
  background: lightblue;
  border-color: white;
  font-weight: bold;
}
.two-axis-table body {
  padding: 1rem;
}
</style>

<div class="ox-hugo-table two-axis-table" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 8</span>:
  Table with 1st row and 1st column highlighted
</div>

| 1 | 2  | 3  | 4  | 5  |
|---|----|----|----|----|
| 2 | 4  | 6  | 8  | 10 |
| 3 | 6  | 9  | 12 | 15 |
| 4 | 8  | 12 | 16 | 20 |
| 5 | 10 | 15 | 20 | 25 |

</div>


### Sane Table {#sane-table}

[Ref](https://css-tricks.com/complete-guide-table-element/#article-header-id-17)

<style>
.sane-table table {
  border-collapse: collapse;
  width: 100%;
}
.sane-table th,
.sane-table td {
  padding: 0.25rem;
  text-align: left;
  border: 1px solid #ccc;
}
</style>

<div class="ox-hugo-table sane-table" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 9</span>:
  Sane Table &#x2014; with minimal styling
</div>

| Name | ID    | Favorite Color |
|------|-------|----------------|
| Jim  | 00001 | Blue           |
| Sue  | 00002 | Red            |
| Barb | 00003 | Green          |

</div>


### Zebra Striping {#zebra-striping}

<style>
.zebra-striping tbody tr:nth-child(odd) {
  background: #eee;
}
</style>

<div class="ox-hugo-table zebra-striping sane-table" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 10</span>:
  Table with zebra striping
</div>

| Last Name | First Name | Email                 | Due     | Web Site                   |
|-----------|------------|-----------------------|---------|----------------------------|
| Smith     | John       | jsmith@gmail.com      | $50.00  | <http://www.jsmith.com>    |
| Bach      | Frank      | fbach@yahoo.com       | $50.00  | <http://www.frank.com>     |
| Doe       | Jason      | jdoe@hotmail.com      | $100.00 | <http://www.jdoe.com>      |
| Conway    | Tim        | tconway@earthlink.net | $50.00  | <http://www.timconway.com> |

</div>


### Highlighting on Hover {#highlighting-on-hover}


#### Highlight Cell {#highlight-cell}

<style>
.hl-table-cell td:hover { /* th:hover also if you wish */
  background: yellow;
}
</style>

<div class="ox-hugo-table hl-table-cell sane-table" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 11</span>:
  Table where the hovered-upon cell highlights
</div>

| Last Name | First Name | Email                 | Due     | Web Site                   |
|-----------|------------|-----------------------|---------|----------------------------|
| Smith     | John       | jsmith@gmail.com      | $50.00  | <http://www.jsmith.com>    |
| Bach      | Frank      | fbach@yahoo.com       | $50.00  | <http://www.frank.com>     |
| Doe       | Jason      | jdoe@hotmail.com      | $100.00 | <http://www.jdoe.com>      |
| Conway    | Tim        | tconway@earthlink.net | $50.00  | <http://www.timconway.com> |

</div>


#### Highlight Row {#highlight-row}

<style>
.hl-table-row tbody tr:hover {
  background: yellow;
}
</style>

<div class="ox-hugo-table hl-table-row sane-table" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 12</span>:
  Table where the hovered-upon row highlights
</div>

| Last Name | First Name | Email                 | Due     | Web Site                   |
|-----------|------------|-----------------------|---------|----------------------------|
| Smith     | John       | jsmith@gmail.com      | $50.00  | <http://www.jsmith.com>    |
| Bach      | Frank      | fbach@yahoo.com       | $50.00  | <http://www.frank.com>     |
| Doe       | Jason      | jdoe@hotmail.com      | $100.00 | <http://www.jdoe.com>      |
| Conway    | Tim        | tconway@earthlink.net | $50.00  | <http://www.timconway.com> |

</div>


### Blur non-hovered Rows {#blur-non-hovered-rows}

_Let's-go-ballistic-with-CSS_ Edition :smile:

<style>
.blur-non-hovered body {
  background: #fafafa;
  color: #444;
  font: 100%/30px 'Helvetica Neue', helvetica, arial, sans-serif;
  text-shadow: 0 1px 0 #fff;
}

.blur-non-hovered strong {
  font-weight: bold;
}

.blur-non-hovered em {
  font-style: italic;
}

.blur-non-hovered table {
  background: #f5f5f5;
  border-collapse: separate;
  box-shadow: inset 0 1px 0 #fff;
  font-size: 12px;
  line-height: 24px;
  margin: 30px auto;
  text-align: left;
  width: 800px;
}

.blur-non-hovered th {
  background: linear-gradient(#777, #444);
  border-left: 1px solid #555;
  border-right: 1px solid #777;
  border-top: 1px solid #555;
  border-bottom: 1px solid #333;
  box-shadow: inset 0 1px 0 #999;
  color: #fff;
  font-weight: bold;
  padding: 10px 15px;
  position: relative;
  text-shadow: 0 1px 0 #000;
}

.blur-non-hovered th:after {
  background: linear-gradient(rgba(255,255,255,0), rgba(255,255,255,.08));
  content: '';
  display: block;
  height: 25%;
  left: 0;
  margin: 1px 0 0 0;
  position: absolute;
  top: 25%;
  width: 100%;
}

.blur-non-hovered th:first-child {
  border-left: 1px solid #777;
  box-shadow: inset 1px 1px 0 #999;
}

.blur-non-hovered th:last-child {
  box-shadow: inset -1px 1px 0 #999;
}

.blur-non-hovered td {
  border-right: 1px solid #fff;
  border-left: 1px solid #e8e8e8;
  border-top: 1px solid #fff;
  border-bottom: 1px solid #e8e8e8;
  padding: 10px 15px;
  position: relative;
  transition: all 300ms;
}

.blur-non-hovered td:first-child {
  box-shadow: inset 1px 0 0 #fff;
}

.blur-non-hovered td:last-child {
  border-right: 1px solid #e8e8e8;
  box-shadow: inset -1px 0 0 #fff;
}

.blur-non-hovered tr:nth-child(odd) td {
  background: #f1f1f1;
}

.blur-non-hovered tr:last-of-type td {
  box-shadow: inset 0 -1px 0 #fff;
}

.blur-non-hovered tr:last-of-type td:first-child {
  box-shadow: inset 1px -1px 0 #fff;
}

.blur-non-hovered tr:last-of-type td:last-child {
  box-shadow: inset -1px -1px 0 #fff;
}

.blur-non-hovered tbody:hover td {
  color: transparent;
  text-shadow: 0 0 3px #aaa;
}

.blur-non-hovered tbody:hover tr:hover td {
  color: #444;
  text-shadow: 0 1px 0 #fff;
}
</style>

<div class="ox-hugo-table blur-non-hovered" markdown="1">
<div></div>
<div class="table-caption">
  <span class="table-number">Table 13</span>:
  Table where rows except the hovered-upon get blurred
</div>

| Last Name | First Name | Email                 | Due     | Web Site                   |
|-----------|------------|-----------------------|---------|----------------------------|
| Smith     | John       | jsmith@gmail.com      | $50.00  | <http://www.jsmith.com>    |
| Bach      | Frank      | fbach@yahoo.com       | $50.00  | <http://www.frank.com>     |
| Doe       | Jason      | jdoe@hotmail.com      | $100.00 | <http://www.jdoe.com>      |
| Conway    | Tim        | tconway@earthlink.net | $50.00  | <http://www.timconway.com> |

</div>

## complete guide

<table>
  <tr>
    <td>This</td>
    <td>Little</td>
    <td>Piggy</td>
    <td>Went</td>
    <td>To</td>
    <td>Market</td>
  </tr>
  <tr>
    <td colspan="2">This</td>
    <td>Little</td>
    <td>Piggy</td>
    <td>Went</td>
    <td>To</td>
  </tr>
    <tr>
    <td colspan="4">This</td>
    <td rowspan="3">Little</td>
    <td>Piggy</td>
  </tr>
  <tr>
    <td rowspan="2">This</td>
    <td>Little</td>
    <td>Piggy</td>
    <td>Went</td>
    <td>To</td>
  </tr>
  <tr>
    <td>Little</td>
    <td>Piggy</td>
    <td>Went</td>
    <td>To</td>
  </tr>
</table>

## cell color

table {
  border-collapse: collapse;
}

td, th {
  border: 5px solid #999;
  padding: 0.5rem;
  text-align: left;
}
<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>ID</th>
      <th>Favorite Color</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Jim</td>
      <td>00001</td>
      <td>Blue</td>
    </tr>
    <tr>
      <td>Sue</td>
      <td>00002</td>
      <td>Red</td>
    </tr>
    <tr>
      <td>Barb</td>
      <td>00003</td>
      <td>Green</td>
    </tr>
  </tbody>
</table>

