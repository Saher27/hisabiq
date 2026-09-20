# Analytics Implementation V1

## Events
1. page_view
2. calculator_start
3. calculator_complete
4. decision_insight_view
5. toolkit_cta_view
6. toolkit_cta_click
7. checkout_start
8. purchase
9. fulfillment_success
10. refund

## Payload
- event_name
- timestamp
- page_version
- anonymous session id
- acquisition source/medium when available
- calculator mode

Do not send raw PII or raw product inputs.