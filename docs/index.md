# Welcome to My eDo Practice Notebook!

This file is a log to note all of the findings and prove that I have practiced doing test cases in eDo.

Goal is to be familiar with the test case and try to learn writing a good documentation as well using MKDocs 🤞.

Environment used during testing is the ***REGRESSION*** environment. However at 2026-06-03, the ***REGRESSION*** environment is under maintenance. So, some of the tests are conducted in ***UAT*** as well.

---

## Test Cases Summary

### Overview

| Metric | Value |
|--------|-------|
| **Total Test Cases** | 10 |
| **Passed** | 10 ✅ |
| **Failed** | 0 ❌ |
| **Success Rate** | 100% |

### Test Case Categories

#### 1. Order Management (3 tests)
- **[E2E_OCP_SF022](test-cases/E2E_OCP_SF022.md)** - Cancel SIM replacement postdating order
- **[SF_E2E_OCP_001](test-cases/SF_E2E_OCP_001.md)** - New connection with prepaid main offer + additional offer
- **[SF_E2E_OCP_018](test-cases/SF_E2E_OCP_018.md)** - SIM replacement order (prepaid & postpaid)

#### 2. Subscriber Lifecycle (4 tests)
- **[SF_E2E_OCP_016](test-cases/SF_E2E_OCP_016.md)** - First Call Activation for inactive prepaid subscriber
- **[SF_E2E_OCP_058](test-cases/SF_E2E_OCP_058.md)** - Active subscriber orders IPP via Modify Offer
- **[SF_E2E_OCP_085](test-cases/SF_E2E_OCP_085.md)** - Change main offer with additional offer (effective immediately)
- **[SF_E2E_OCP_159](test-cases/SF_E2E_OCP_159.md)** - Reconnection of terminated postpaid subscriber with new offer

#### 3. Customer Operations (2 tests)
- **[SF_E2E_OCP_126](test-cases/SF_E2E_OCP_126.md)** - Transfer ownership for postpaid subscriber
- **[SF_E2E_OCP_188](test-cases/SF_E2E_OCP_188.md)** - Starter pack activation for existing customer

#### 4. Validation & Edge Cases (1 test)
- **[SF_E2E_OCP_162](test-cases/SF_E2E_OCP_162.md)** - Reconnection failure after X days (>180 days limit)

### Key Features Tested

✅ **Prepaid Services**: New connections, activations, IPP modifications, balance payments  
✅ **Postpaid Services**: SIM replacement, offer changes, ownership transfer, reconnections  
✅ **Order Processing**: Creation, review, payment, submission, provisioning, completion  
✅ **Validation Rules**: File size limits (5MB), ICCID states, subscriber status checks, overdue amounts  
✅ **Notifications**: SMS/email delivery, CIC whitelist management  
✅ **Documentation**: e-Receipt, e-Request Form (e-RF), POS receipts, eSIM QR codes  
✅ **Database Verification**: SQL queries for `ic_acc_nbr`, `ic_sim_card`, `spn.sa_order` tables  

### Common Patterns Observed

1. **Order Flow**: Order Entry → Retrieve Customer → Select Operation → Review → T&C → Payment → Submission → Completion
2. **Pre-validation**: System checks pending orders, overdue amounts, subscriber status before proceeding
3. **Payment Methods**: Cash (service number only) vs Balance (service number + account info)
4. **State Transitions**: Proper tracking of subscriber/SIM card states (Available → Locked → In Use)
5. **Evidence Collection**: Screenshots at each step + database verification for critical state changes

### Lessons Learned

- **Attachment Upload**: Files >5MB cause system stall without clear error messages
- **CIC Whitelist**: Customer contacts must be whitelisted for successful notification delivery
- **ICCID States**: Automatic state changes occur (Available → Locked upon input → In Use after completion)
- **Provisioning Tracking**: Order provisioning states can be queried from `spn.sa_order` table
- **Termination Limits**: Reconnection not allowed after 180 days from termination date

### Database Queries Reference

```sql
-- Check service number state
SELECT prefix || acc_nbr AS service_nbr, hlr_id, state_date, acc_nbr_state, acc_nbr_state_name
FROM ic_acc_nbr
INNER JOIN ic_acc_nbr_state USING (acc_nbr_state)
WHERE prefix || acc_nbr = '<MSISDN>';

-- Check SIM card state
SELECT iccid, hlr_id, state_date, sim_state, sim_state_name
FROM ic_sim_card
INNER JOIN ic_sim_state USING (sim_state)
WHERE iccid='<ICCID>';

-- Check order provisioning state
SELECT order_item_id, state_date, created_date, state, order_state_name
FROM spn.sa_order
INNER JOIN spn.sa_order_state ON spn.sa_order.state = spn.sa_order_state.order_state
WHERE order_item_id=<order_id>;
```

---

*Last Updated: June 4, 2026*
