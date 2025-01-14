# /tests: Unit Tests for HL7-to-FHIR Transformation

This document provides instructions for setting up and running unit tests for HL7-to-FHIR transformation. It includes details on tools, steps for testing using HAPI FHIR and Postman, and sample HL7 and FHIR resources.

---

## Tools Required

### 1. **Python and Pytest**
- Install Python (>=3.8) and the `pytest` library for running automated unit tests.
- Install the `hl7apy` and `requests` libraries for HL7 parsing and API interaction:
  ```bash
  pip install pytest hl7apy requests
  ```

### 2. **HAPI FHIR Server**
- Download and run the [HAPI FHIR Server](https://hapifhir.io/):
  ```bash
  java -jar hapi-fhir-jpaserver-example-*.jar
  ```
- Use the default base URL: `http://localhost:8080/fhir`.

### 3. **Postman**
- Use [Postman](https://www.postman.com/) to manually test FHIR resources by sending requests to the HAPI FHIR server.

---

## Testing Setup

### Directory Structure
```
/tests
  |-- test_hl7_to_fhir.py   # Unit test script
  |-- sample_hl7.hl7        # Sample HL7 message
  |-- expected_fhir.json    # Expected FHIR resource
```

---

## Sample Fake Data

### HL7 Message: `sample_hl7.hl7`
```hl7
MSH|^~\&|SendingApp|SendingFac|ReceivingApp|ReceivingFac|202301011230||ADT^A01|12345|P|2.5
PID|1||123456^^^MRN||Doe^John||19820515|M||||||||||123456789
IN1|1|Aetna|123456|Group Health Plan|||||||Doe^John
```

### Corresponding FHIR Resource: `expected_fhir.json`
```json
{
  "resourceType": "Coverage",
  "id": "example-coverage",
  "status": "active",
  "beneficiary": {
    "reference": "Patient/123456"
  },
  "payor": [
    {
      "display": "Aetna"
    }
  ],
  "group": "Group Health Plan"
}
```

---

## Unit Test Script: `test_hl7_to_fhir.py`

### Test File
```python
import pytest
from hl7apy import parser
from hl7_to_fhir import hl7_to_fhir_transform
import requests

FHIR_BASE_URL = "http://localhost:8080/fhir"

# Sample HL7 and expected FHIR files
HL7_MESSAGE_PATH = "./sample_hl7.hl7"
EXPECTED_FHIR_PATH = "./expected_fhir.json"

def test_hl7_to_fhir_conversion():
    # Load HL7 message
    with open(HL7_MESSAGE_PATH, "r") as hl7_file:
        hl7_message = hl7_file.read()

    # Perform transformation
    fhir_resource = hl7_to_fhir_transform(hl7_message)

    # Load expected FHIR resource
    with open(EXPECTED_FHIR_PATH, "r") as fhir_file:
        expected_fhir = fhir_file.read()

    # Validate the transformation
    assert fhir_resource == expected_fhir

def test_fhir_server_integration():
    # Load expected FHIR resource
    with open(EXPECTED_FHIR_PATH, "r") as fhir_file:
        fhir_resource = fhir_file.read()

    # POST the resource to the FHIR server
    response = requests.post(f"{FHIR_BASE_URL}/Coverage", json=fhir_resource)
    
    # Validate the response
    assert response.status_code == 201
    posted_resource = response.json()
    assert posted_resource["resourceType"] == "Coverage"
    assert posted_resource["status"] == "active"
```

---

## Steps to Test Using HAPI FHIR and Postman

### Using HAPI FHIR Server
1. **Start the Server:**
   ```bash
   java -jar hapi-fhir-jpaserver-example-*.jar
   ```
2. **Run Unit Tests:**
   ```bash
   pytest test_hl7_to_fhir.py
   ```
   Ensure all tests pass successfully.

### Using Postman
1. **Create a New Request:**
   - Set the method to `POST`.
   - Enter the URL: `http://localhost:8080/fhir/Coverage`.
   - Add the `Content-Type: application/json` header.
2. **Add the Payload:**
   - Paste the content of `expected_fhir.json` into the request body.
3. **Send the Request:**
   - Verify the response status is `201 Created`.
   - Check the returned FHIR resource for accuracy.

---

## Notes
- Update the `hl7_to_fhir_transform` function with the logic specific to your transformation requirements.
- Expand the `test_hl7_to_fhir.py` file to include additional test cases for edge scenarios (e.g., missing fields, invalid formats).

---

By following this documentation, you can effectively test and validate HL7-to-FHIR transformations, ensuring compliance and reliability in your payer workflows.
