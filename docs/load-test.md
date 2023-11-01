## Load Testing

#### Setup Load Test



```bash
kubectl scale --replicas=10 deployment.apps/order-service -n pets

# single order manually
curl -i http://192.168.1.1:3000/ -X POST -H 'Content-Type: application/json' -d '{"customerId": "1234567890", "items": [{"productId": 1,"quantity": 5,"price": 9.99},{"productId": 10,"quantity": 2,"price": 5.99}]}' 

# loop orders
while true; do curl -i http://192.168.1.1:3000/ -X POST -H 'Content-Type: application/json' -d '{"customerId": "1234567890", "items": [{"productId": 1,"quantity": 5,"price": 9.99},{"productId": 10,"quantity": 2,"price": 5.99}]}' && echo '' ; sleep 1; done

kubectl scale --replicas=1 deployment.apps/order-service -n pets
```