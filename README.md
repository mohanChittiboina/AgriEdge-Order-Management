# 🌾 AgriEdge Order & Shipment Management System

A Salesforce-powered solution tailored for the agricultural sector—designed to automate order processing, order item management, and shipment tracking using custom Salesforce objects and Apex logic.

---

## 🚀 Features

- **Custom Objects**  
  - `AgriEdge_Order__c` – captures order data  
  - `AgriEdge_OrderItem__c` – manages associated line items  
  - `AgriEdge_Shipment__c` – handles shipment tracking and updates  

- **Automation & Business Logic**  
  - Order totals calculated automatically via `OrderTotalUpdater` Apex class  
  - Dynamic updates of **Order Status** and **Payment Status** via triggers  
  - Shipment creation and validation (e.g., require tracking numbers)  

- **Data Integrity & Validation**  
  - Enforced through validation rules and lookup relationships  
  - Prevents incomplete or inconsistent records  

- **User Interface**  
  - Lightning UI layouts for intuitive record interaction  
  - Option to build mobile and portal interfaces using LWC and Experience Cloud  

---

## 🛠️ Architecture Overview

| Layer            | Description                                                                                 |
|------------------|---------------------------------------------------------------------------------------------|
| **Salesforce CRM** | Foundation platform with metadata-driven custom objects and page layouts                    |
| **Apex Logic**     | Triggers and helper classes: `OrderTotalUpdater`, `OrderStatusUpdater`, `ShipmentHandler` |
| **Automation Tools**| Validation Rules, Invocable Apex, SOQL, Lookup Relationships                               |
| **User Interface** | Lightning layouts arranged for efficient order/shipment workflows                           |

---

## 📁 Project Structure

AgriEdge-Order-Management/
│
├── force-app/
│ └── main/
│ └── default/
│ ├── classes/
│ ├── triggers/
│ ├── objects/
│ └── layouts/
│
├── screenshots/
├── README.md
├── .gitignore
├── sfdx-project.json
└── config/

yaml
Copy
Edit

## 🪄 Usage / Deployment

1. Clone the repository:
   ```bash
   git clone https://github.com/mohanChittiboina/AgriEdge-Order-Management.git
Set up SFDX:

bash
Copy
Edit
sfdx force:auth:web:login -a DevHub
sfdx force:org:create -f config/project-scratch-def.json -a AgriEdgeDev
sfdx force:source:push -u AgriEdgeDev
Assign necessary permissions:

Profiles and permission sets for admin, sales, and logistics roles

Load sample data using DataImport or Apex classes

🎯 Roadmap / Future Enhancements
Real-time inventory sync via AgriEdge_Inventory__c

Customer portal via Experience Cloud for self-service

Mobile UI using LWC or Salesforce Mobile App

Dynamic dashboards with Power BI or Einstein Analytics

Carrier integration (FedEx, DHL) for live tracking

AI-powered recommendations using Salesforce Einstein

👤 Developed By
Mohan Chittiboina
AgriEdge Pvt. Ltd.
LinkedIn Profile
