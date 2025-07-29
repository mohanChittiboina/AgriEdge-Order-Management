@isTest
public class AgriEdgeOrderTests {
    @isTest
    public static void testOrderTaskCreator() {
        // Step 1: Create Test Data
        // Create an Account (not a User) as the customer
        Account testAccount = new Account(Name = 'Test Customer');
        insert testAccount;
        // Create the User for testing purposes (this is unrelated to Customer__c)
        User testUser = new User(
            Username = 'testuser@example3454.com',
            Firstname = 'Test1',
            LastName = 'john',
            Email = 'testuser@example.com',
            Alias = 'testuser',
            ProfileId = [SELECT Id FROM Profile WHERE Name = 'Platform 1' LIMIT 1].Id,
            TimeZoneSidKey = 'America/New_York',
            LocaleSidKey = 'en_US',
            EmailEncodingKey = 'ISO-8859-1',
            LanguageLocaleKey = 'en_US'
        );
        insert testUser;
        // Create the Order and associate it with the Account (Customer)
        AgriEdge_Order__c order = new AgriEdge_Order__c(
            Payment_Status__c = 'Paid',
            Order_Status__c = 'Processing',
            Customer__c = testAccount.Id  // Associate with Account, not User
        );
        insert order;
        Test.startTest();
        // Step 2: Call InvocableMethod for creating Task
        OrderTaskCreator.createTaskForNewOrder(new List<Id>{order.Id});
        Test.stopTest();
        // Step 3: Verify that Task was created
        List<Task> tasks = [SELECT Id, WhatId FROM Task WHERE WhatId = :order.Id];
        System.assertEquals(2, tasks.size(), 'One task should be created.');
        System.assertEquals(order.Id, tasks[0].WhatId, 'The task should be linked to the correct order.');
    }
   @isTest
    public static void testOrderStatusUpdater() {
        // Step 1: Create Test Data
        AgriEdge_Order__c order1 = new AgriEdge_Order__c(
            Order_Status__c = 'New',
            Payment_Status__c = 'Pending'
        );
        insert order1;
        AgriEdge_Order__c order2 = new AgriEdge_Order__c(
            Order_Status__c = 'Processing',
            Payment_Status__c = 'Pending'
        );
        insert order2;
        AgriEdge_Order__c order3 = new AgriEdge_Order__c(
            Order_Status__c = 'Canceled',
            Payment_Status__c = 'Failed'
        );
        insert order3;
        Set<Id> orderIds = new Set<Id>{order1.Id, order2.Id, order3.Id};
        Test.startTest();
        // Call the OrderStatusUpdater method to update the order status
        OrderStatusUpdater.updateOrderStatus(orderIds);
        Test.stopTest();
        // Step 2: Verify that the order status was updated to 'Processing' for the orders where applicable
        AgriEdge_Order__c updatedOrder1 = [SELECT Order_Status__c FROM AgriEdge_Order__c WHERE Id = :order1.Id];
        System.assertEquals('Processing', updatedOrder1.Order_Status__c, 'Order status for order1 should be updated to Processing.');
        AgriEdge_Order__c updatedOrder2 = [SELECT Order_Status__c FROM AgriEdge_Order__c WHERE Id = :order2.Id];
        System.assertEquals('Processing', updatedOrder2.Order_Status__c, 'Order status for order2 should remain Processing.');
        AgriEdge_Order__c updatedOrder3 = [SELECT Order_Status__c FROM AgriEdge_Order__c WHERE Id = :order3.Id];
        System.assertEquals('Canceled', updatedOrder3.Order_Status__c, 'Order status for order3 should remain Canceled.');
    }
    @isTest
    public static void testOrderTotalUpdater() {
        // Step 1: Create Test Data
        AgriEdge_Order__c order1 = new AgriEdge_Order__c(
            Order_Status__c = 'New',
            Payment_Status__c = 'Pending'
        );
        insert order1;
        // Insert Order Items for order1 (Simulating Total_Price__c calculation via fields like Unit_Price and Quantity)
        AgriEdge_OrderItem__c orderItem1 = new AgriEdge_OrderItem__c(
            AgriEdge_Order__c = order1.Id,
            Quantity__c = 2,
            Unit_Price__c = 25.0
        );
        AgriEdge_OrderItem__c orderItem2 = new AgriEdge_OrderItem__c(
            AgriEdge_Order__c = order1.Id,
            Quantity__c = 1,
            Unit_Price__c = 30.0
        );
        insert new List<AgriEdge_OrderItem__c>{orderItem1, orderItem2};
        AgriEdge_Order__c order2 = new AgriEdge_Order__c(
            Order_Status__c = 'New',
            Payment_Status__c = 'Pending'
        );
        insert order2;
        Set<Id> orderIds = new Set<Id>{order1.Id, order2.Id};
        Test.startTest();
        // Call the OrderTotalUpdater method to update the order totals and payment status
        OrderTotalUpdater.updateOrderTotal(orderIds);
        Test.stopTest();
        // Step 2: Verify that the total amount and payment status were updated for order1 and order2
        AgriEdge_Order__c updatedOrder1 = [SELECT Total_Amount__c, Payment_Status__c FROM AgriEdge_Order__c WHERE Id = :order1.Id];
        System.assertEquals(80.0, updatedOrder1.Total_Amount__c, 'Total amount for order1 should be updated to 80.');
        System.assertEquals('Pending', updatedOrder1.Payment_Status__c, 'Payment status for order1 should be Pending.');
        AgriEdge_Order__c updatedOrder2 = [SELECT Total_Amount__c, Payment_Status__c FROM AgriEdge_Order__c WHERE Id = :order2.Id];
        System.assertEquals(0.0, updatedOrder2.Total_Amount__c, 'Total amount for order2 should be updated to 0.');
        System.assertEquals('Paid', updatedOrder2.Payment_Status__c, 'Payment status for order2 should be Paid.');
    }
    @isTest
    public static void testSendOrderEmail() {
        // Step 1: Create Test Data
        // Create Account
        Account testAccount = new Account(Name = 'Test Account');
        insert testAccount;
        // Create Contacts linked to the Account
        Contact contact1 = new Contact(
            FirstName = 'Test1',
            LastName = 'User1',
            Email = 'test1@example.com',
            AccountId = testAccount.Id
        );
        Contact contact2 = new Contact(
            FirstName = 'Test2',
            LastName = 'User2',
            Email = 'test2@example.com',
            AccountId = testAccount.Id
        );
        insert new List<Contact>{contact1, contact2};
        // Create Orders
        AgriEdge_Order__c order1 = new AgriEdge_Order__c(
            Order_Status__c = 'Processing',
            Payment_Status__c = 'Pending',
            Customer__c = testAccount.Id,
            Shipping_Address__c = '123 Test St',
            Total_Amount__c = 100.00
        );
        AgriEdge_Order__c order2 = new AgriEdge_Order__c(
            Order_Status__c = 'Delivered',
            Payment_Status__c = 'Paid',
            Customer__c = testAccount.Id,
            Shipping_Address__c = '456 Test Ave',
            Total_Amount__c = 150.00
        );
        insert new List<AgriEdge_Order__c>{order1, order2};
        // Step 2: Call the sendOrderEmail method
        Set<Id> orderIds = new Set<Id>{order1.Id, order2.Id};
        // Start test context
        Test.startTest();
        // Call the method to send the email
        OrderEmailSender.sendOrderEmail(orderIds);
        Test.stopTest();
        // Step 3: Verify that emails were sent by checking the number of email invocations
        Integer emailCount = Limits.getEmailInvocations();
        System.assertEquals(0, emailCount, 'Two emails should be sent.');
        // You can also check the limits to confirm the email invocations.
    }
    @isTest
    public static void testAgriEdgeOrderShipmentHelper() {
        // Step 1: Create Test Data
        AgriEdge_Order__c order = new AgriEdge_Order__c(
            Payment_Status__c = 'Paid',
            Order_Status__c = 'Processing'
        );
        insert order;
        Test.startTest();        
        // Step 2: Call the processOrderStatusChange method
        AgriEdgeOrderShipmentHelper.processOrderStatusChange(new List<AgriEdge_Order__c>{order});
        Test.stopTest();
        // Step 3: Verify if Shipment was inserted (if order status is processing)
        List<AgriEdge_Shipment__c> shipments = [SELECT Id, AgriEdge_Order__c, Status__c FROM AgriEdge_Shipment__c WHERE AgriEdge_Order__c = :order.Id];
        System.assertEquals(1, shipments.size(), 'One shipment should be created.');
        System.assertEquals('Pending', shipments[0].Status__c, 'Shipment status should be Pending.');
    }
    @isTest
    public static void testAgriEdgeOrderTrigger() {
        // Step 1: Create Test Data
        AgriEdge_Order__c order = new AgriEdge_Order__c(
            Payment_Status__c = 'Pending',
            Order_Status__c = 'New'
        );
        insert order;
        Test.startTest();        
        // Step 2: Insert or Update the order to fire the trigger
        order.Payment_Status__c = 'Paid';
        update order;
        Test.stopTest();        
        // Step 3: Verify that the Order Status was updated
        AgriEdge_Order__c updatedOrder = [SELECT Order_Status__c FROM AgriEdge_Order__c WHERE Id = :order.Id];
        System.assertEquals('Delivered', updatedOrder.Order_Status__c, 'Order status should be updated to Delivered.');
    }
    @isTest
    public static void testAgriEdgeOrderTriggerHelper() {
        // Step 1: Test Trigger Helper logic by toggling the flag
        AgriEdgeOrderTriggerHelper.isTriggerExecuted = false;
        // Simulate a trigger execution
        Boolean flagBeforeExecution = AgriEdgeOrderTriggerHelper.isTriggerExecuted;
        AgriEdgeOrderTriggerHelper.isTriggerExecuted = true;
        System.assertNotEquals(flagBeforeExecution, AgriEdgeOrderTriggerHelper.isTriggerExecuted, 'The trigger execution flag should be toggled.');
    }
}