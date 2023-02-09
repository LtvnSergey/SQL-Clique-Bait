# Project: Clique Bait

<img src="https://user-images.githubusercontent.com/35038779/216983985-f8472c2f-3dda-4ab8-8a9c-1fbe237ff00d.png" alt="Image" width="500" height="520">

# Overview
The goal of the project is to analyse and calculate funnal fallout rates for Clique Bait online seafood store. 

The project is based on [SQL-8-week-challenge](https://8weeksqlchallenge.com/case-study-6/) case. 

## Contents

- [Project workflow](#project-workflow)
- [Data description](#data-description)
- [Modules and tools](#modules-and-tools)


### Project workflow

  1. Setting up PostgresSQL server 
  2. [Create project database and load schemas with input tables](https://github.com/LtvnSergey/SQL-Clique-Bait/blob/main/input.md)
  3. [Digit analysis](https://github.com/LtvnSergey/SQL-Clique-Bait/blob/main/analysis/digital_analysis.md)
  4. [Product funnel analysis](https://github.com/LtvnSergey/SQL-Clique-Bait/blob/main/analysis/product_funnel_analysis.md)
  5. [Campaigns analysis](https://github.com/LtvnSergey/SQL-Clique-Bait/blob/main/analysis/campaigns_analysis.md)


### Data description

* **Users**

Customers who visit the Clique Bait website are tagged via their `cookie_id`.

![image](https://user-images.githubusercontent.com/35038779/217028827-09b348c0-737f-47fc-9ae8-f27eb7c8f73c.png)

* **Events**

Customer visits are logged in this events table at a `cookie_id` level and the `event_type` and `page_id` values can be used to join onto relevant satellite tables to obtain further information about each event.

The sequence_number is used to order the events within each visit.

![image](https://user-images.githubusercontent.com/35038779/217029123-2695dc66-1b27-42b1-a7e5-b199a0e55aaf.png)


* **Event Identifier** 

The `event_identifier` table shows the types of events which are captured by Clique Bait’s digital data systems.

![image](https://user-images.githubusercontent.com/35038779/217029809-bbfca0d9-0f60-4f2a-8235-6fd3e816da02.png)


* **Campaign Identifier**

This table shows information for the 3 campaigns that Clique Bait has ran on their website so far in 2020.

![image](https://user-images.githubusercontent.com/35038779/217030051-2493d98d-4ccf-4fd6-9adf-6e9804313f2f.png)


* **Page Hierarchy**

This table lists all of the pages on the Clique Bait website which are tagged and have data passing through from user interaction events.

![image](https://user-images.githubusercontent.com/35038779/217030242-6d7a50ac-dabb-4948-a207-131ec1424406.png)



* **Entity Relationship Diagram**:

<img src="https://user-images.githubusercontent.com/35038779/217026508-fbcf5de1-463b-4450-8dd4-9c07aeaac714.png" alt="Image" width="1500" height="520">


### Modules and tools

PostgresSQL | PgAdmin 4
