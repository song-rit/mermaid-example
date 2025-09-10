```mermaid
stateDiagram-v2
    [*] --> CartCreated : Customer adds items
    
    state "Shopping Cart" as cart {
        CartCreated --> ItemsAdded : add_item()
        ItemsAdded --> ItemsAdded : add_item() / update_quantity()
        ItemsAdded --> ItemsRemoved : remove_item()
        ItemsRemoved --> ItemsAdded : add_item()
        ItemsRemoved --> CartAbandoned : timeout(30min)
        ItemsAdded --> CartAbandoned : timeout(30min)
        
        state ItemsAdded {
            [*] --> ValidatingInventory
            ValidatingInventory --> InStock : items_available
            ValidatingInventory --> OutOfStock : items_unavailable
            OutOfStock --> InStock : restock_notification
            InStock --> PriceCalculated : calculate_total()
        }
    }
    
    cart --> CheckoutInitiated : checkout_clicked
    CartAbandoned --> [*] : cleanup_cart()
    
    state "Checkout Process" as checkout {
        CheckoutInitiated --> CustomerInfo : enter_details()
        CustomerInfo --> PaymentInfo : validate_customer()
        CustomerInfo --> CustomerInfo : validation_failed
        
        PaymentInfo --> PaymentValidation : submit_payment()
        PaymentValidation --> PaymentApproved : payment_success
        PaymentValidation --> PaymentDeclined : payment_failed
        PaymentDeclined --> PaymentInfo : retry_payment
        PaymentDeclined --> PaymentAbandoned : max_retries_reached
        
        PaymentApproved --> OrderConfirmed : create_order()
    }
    
    checkout --> CartAbandoned : cancel_checkout
    PaymentAbandoned --> CartAbandoned
    
    state "Order Processing" as processing {
        OrderConfirmed --> InventoryReserved : reserve_items()
        InventoryReserved --> InventoryConfirmed : inventory_check_passed
        InventoryReserved --> InventoryFailed : inventory_insufficient
        InventoryFailed --> OrderCancelled : refund_payment()
        
        InventoryConfirmed --> OrderQueued : add_to_fulfillment_queue()
        OrderQueued --> OrderProcessing : worker_picks_order()
        OrderProcessing --> ItemsPicked : pick_items()
        ItemsPicked --> QualityCheck : quality_control()
        
        state QualityCheck {
            [*] --> Inspecting
            Inspecting --> QCPassed : items_ok
            Inspecting --> QCFailed : items_defective
            QCFailed --> ItemsReplaced : replace_items()
            ItemsReplaced --> Inspecting
            QCPassed --> [*]
        }
        
        QualityCheck --> OrderPacked : items_approved
        OrderPacked --> ShippingLabelCreated : generate_label()
        ShippingLabelCreated --> ReadyForShipment : move_to_shipping_dock()
    }
    
    state "Shipping & Delivery" as shipping {
        ReadyForShipment --> InTransit : carrier_pickup()
        
        state InTransit {
            [*] --> PickedUp
            PickedUp --> InTransitToHub : scan_at_origin
            InTransitToHub --> AtSortingFacility : arrive_at_hub
            AtSortingFacility --> OutForDelivery : sorted_for_delivery
            OutForDelivery --> AtDeliveryLocation : arrive_at_destination
            
            state "Delivery Attempts" as delivery_attempts {
                AtDeliveryLocation --> DeliveryAttempt1 : first_attempt
                DeliveryAttempt1 --> Delivered : successful_delivery
                DeliveryAttempt1 --> DeliveryAttempt2 : delivery_failed
                DeliveryAttempt2 --> Delivered : successful_delivery  
                DeliveryAttempt2 --> DeliveryAttempt3 : delivery_failed
                DeliveryAttempt3 --> Delivered : successful_delivery
                DeliveryAttempt3 --> ReturnToSender : final_attempt_failed
            }
        }
        
        InTransit --> DeliveryException : package_damaged / lost
        DeliveryException --> InvestigationStarted : file_claim()
        InvestigationStarted --> RefundIssued : investigation_complete
        InvestigationStarted --> ReplacementSent : send_replacement
    }
    
    state "Post-Delivery" as post_delivery {
        Delivered --> DeliveryConfirmed : customer_confirms
        Delivered --> DeliveryDisputed : customer_disputes(48h)
        
        DeliveryConfirmed --> OrderCompleted : satisfaction_survey()
        DeliveryDisputed --> InvestigationStarted : investigate_dispute()
        
        OrderCompleted --> ReturnRequested : return_window_open(30days)
        ReturnRequested --> ReturnApproved : validate_return_request()
        ReturnApproved --> ReturnInProgress : customer_ships_back()
        ReturnInProgress --> ReturnReceived : package_received()
        ReturnReceived --> RefundProcessed : inspect_returned_items()
        ReturnReceived --> ReturnRejected : items_not_returnable()
        
        OrderCompleted --> ReviewSubmitted : customer_leaves_review()
    }
    
    state "Terminal States" as terminal {
        OrderCancelled --> [*] : order_cleanup()
        RefundIssued --> [*] : case_closed()
        RefundProcessed --> [*] : transaction_complete()
        ReturnToSender --> [*] : return_to_inventory()
        ReturnRejected --> [*] : return_shipped_back()
        ReviewSubmitted --> [*] : order_archived()
        ReplacementSent --> processing : create_replacement_order()
    }
    
    %% Concurrent states for customer service
    state CustomerService {
        [*] --> Available
        Available --> InquiryReceived : customer_contact()
        InquiryReceived --> ResolvingIssue : agent_assigned()
        ResolvingIssue --> IssueResolved : resolution_provided()
        IssueResolved --> Available : case_closed()
        ResolvingIssue --> EscalatedToManager : escalation_required()
        EscalatedToManager --> ResolvingIssue : manager_provides_solution()
    }
    
    %% Global transitions that can happen from multiple states
    processing --> OrderCancelled : customer_cancels() [before_shipping]
    shipping --> OrderCancelled : customer_cancels() [with_fee]
    
    %% Error handling
    state ErrorHandling {
        [*] --> SystemError
        SystemError --> ErrorLogged : log_error()
        ErrorLogged --> NotificationSent : alert_support()
        NotificationSent --> [*] : error_resolved()
    }
    
    %% Notes for complex transitions
    note right of CartAbandoned
        Cleanup occurs after 30min timeout
        or explicit abandonment
    end note
    
    note right of PaymentDeclined
        Max 3 retry attempts allowed
        Different decline reasons handled
    end note
    
    note right of InTransit
        Real-time tracking updates
        Multiple carrier integrations
    end note