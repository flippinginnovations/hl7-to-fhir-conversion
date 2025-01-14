# Python Scripts for HL7 to FHIR Conversion and Validation

## 1. Convert HL7 to FHIR
This script parses an HL7 message and maps it to a FHIR resource. It includes an example for converting a PID segment to a FHIR Patient resource.

```python
import json
from hl7apy.parser import parse_message

# Sample HL7 message
hl7_message = """\
MSH|^~\&|SendingApp|SendingFac|ReceivingApp|ReceivingFac|202301011230||ADT^A01|12345|P|2.5\n
PID|1||123456^^^MRN||Doe^John||19820515|M||||||||||123456789\n
"""

def hl7_to_fhir(hl7):
    # Parse HL7 message
    parsed_message = parse_message(hl7)

    # Extract PID segment
    pid = parsed_message.segment('PID')

    # Map to FHIR Patient resource
    fhir_patient = {
        "resourceType": "Patient",
        "id": pid[3][0],  # MRN
        "name": [
            {
                "use": "official",
                "family": pid[5][0][0],  # Last Name
                "given": [pid[5][0][1]]  # First Name
            }
        ],
        "gender": "male" if pid[8] == "M" else "female",
        "birthDate": f"{pid[7][:4]}-{pid[7][4:6]}-{pid[7][6:]}"
    }

    return fhir_patient

# Convert HL7 to FHIR
fhir_resource = hl7_to_fhir(hl7_message)
print(json.dumps(fhir_resource, indent=4))
```

## 2. Validate FHIR Resource
This script validates a FHIR resource using the FHIR Validator API.

```python
import requests

FHIR_VALIDATOR_URL = "http://validator.fhir.org/validator"

# Sample FHIR resource
fhir_resource = {
    "resourceType": "Patient",
    "id": "example",
    "name": [
        {
            "use": "official",
            "family": "Doe",
            "given": ["John"]
        }
    ],
    "gender": "male",
    "birthDate": "1982-01-23"
}

# Validate FHIR resource
def validate_fhir(resource):
    response = requests.post(FHIR_VALIDATOR_URL, json=resource)
    if response.status_code == 200:
        print("Validation Results:", response.json())
    else:
        print("Validation failed with status code:", response.status_code)

# Validate the resource
validate_fhir(fhir_resource)
```

## How to Use
1. Place the scripts in the `/scripts` folder of the repository.
2. Install dependencies:
    ```bash
    pip install requests hl7apy
    ```
3. Run the scripts to convert HL7 messages to FHIR resources and validate them using the FHIR Validator API.
