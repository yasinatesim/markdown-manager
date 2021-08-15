<h3 align="center">
  <br />
   <a  href="<%= GITHUB_URL =%>"><img src="<%= PROJECT_LOGO =%>" alt="<%= PROJECT_NAME =%>" width="200" /></a>
  <br />
<%= PROJECT_NAME =%>
  <br />
</h3>

<hr />

<p  align="center"><%= PROJECT_DESCRIPTION =%></p>


  <p align="center">
<% @foreach item in links %>
· <a  href="<%= item.link =%>"><%= item.label =%></a>
<% @endforeach %>
  </p>

## 📖 About

<%= PROJECT_ABOUT =%>

### 📚Tech Stack

<table>
<% @foreach tech in techStacks %>
  <tr>
    <td> <a href="<%= tech.url =%>"><%= tech.name =%></a></td>
    <td><%= tech.description =%></td>
  </tr>
<% @endforeach %>
</table>

## 🧐 What's inside?

<%= WHATS_INSIDE =%>

## Getting Started

### 📦 Prerequisites

<%= PREREQUISITES =%>

### ⚙️ How To Use

<%= HOW_TO_USE =%>

## 🔑 License

* Copyright © <%= YEAR =%> - <%= LICENSE_NAME =%> License.

See [LICENSE](<%= GITHUB_URL =%>/blob/<%= GITHUB_MAIN_BRANCH =%>/LICENSE) for more information.
