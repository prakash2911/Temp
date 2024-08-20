curl --cert /path/to/your/certificate.p12:mypassword --cert-type P12 \
     --header "Content-Type: application/json" \
     --request POST \
     --data '{"username":"testuser", "password":"testpass"}' \
     https://api.example.com/login
