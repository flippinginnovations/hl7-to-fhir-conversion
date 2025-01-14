# /examples: Sample HL7 Messages and Converted FHIR Resources

## Da Vinci Use Case Examples
This directory contains sample HL7 messages and their corresponding converted FHIR resources for key Da Vinci use cases:

---

### 🏥 **CRD (Coverage Requirements Discovery)**
**Objective:** Simplify eligibility checks and coverage requirements.

#### Example HL7 Message (Eligibility Inquiry):
```hl7
MSH|^~\&|SendingApp|SendingFac|ReceivingApp|ReceivingFac|202301011230||ADT^A01|12345|P|2.5
PID|1||123456^^^MRN||Doe^John||19820515|M||||||||||123456789
IN1|1|Aetna|123456|Group Health Plan
```

#### Converted FHIR Resources:
**EligibilityRequest:**
```json
{
    "resourceType": "EligibilityRequest",
    "id": "example-crd",
    "status": "active",
    "priority": {
        "coding": [
            {
                "system": "http://terminology.hl7.org/CodeSystem/processpriority",
                "code": "normal"
            }
        ]
    },
    "patient": {
        "reference": "Patient/12345"
    },
    "insurance": [
        {
            "coverage": {
                "reference": "Coverage/123456"
            }
        }
    ]
}
```

**EligibilityResponse:**
```json
{
    "resourceType": "EligibilityResponse",
    "id": "example-response",
    "status": "active",
    "outcome": "complete",
    "patient": {
        "reference": "Patient/12345"
    },
    "insurance": [
        {
            "coverage": {
                "reference": "Coverage/123456"
            },
            "inforce": true
        }
    ]
}
```

---

### ✅ **PAS (Prior Authorization Support)**
**Objective:** Automate and accelerate prior authorization workflows.

#### Example HL7 Message (Prior Authorization Request):
```hl7
PID|1||123456^^^MRN||Doe^John||19820515|M
IN1|1|Aetna|123456|Group Health Plan
OBR|1|||Procedure^123|||20230101
```

#### Converted FHIR Resources:
**PriorAuthorizationRequest:**
```json
{
    "resourceType": "PriorAuthorizationRequest",
    "id": "example-pas",
    "status": "active",
    "patient": {
        "reference": "Patient/12345"
    },
    "item": [
        {
            "service": {
                "coding": [
                    {
                        "system": "http://example.org/procedures",
                        "code": "123",
                        "display": "Procedure"
                    }
                ]
            }
        }
    ]
}
```

**PriorAuthorizationResponse:**
```json
{
    "resourceType": "PriorAuthorizationResponse",
    "id": "response-pas",
    "status": "active",
    "outcome": "approved",
    "patient": {
        "reference": "Patient/12345"
    }
}
```

---

### 🔄 **PDex (Payer Data Exchange)**
**Objective:** Enable seamless data exchange for value-based care.

#### Example HL7 Message (Claim Information):
```hl7
PID|1||123456^^^MRN||Doe^John||19820515|M
IN1|1|Aetna|123456|Group Health Plan
FT1|1|||20230101|20230101|500|USD|CPT:12345|Service
```

#### Converted FHIR Resources:
**ExplanationOfBenefit:**
```json
{
    "resourceType": "ExplanationOfBenefit",
    "id": "example-pdex",
    "status": "active",
    "patient": {
        "reference": "Patient/12345"
    },
    "total": {
        "value": 500,
        "currency": "USD"
    },
    "item": [
        {
            "service": {
                "coding": [
                    {
                        "system": "http://www.ama-assn.org/go/cpt",
                        "code": "12345",
                        "display": "Service"
                    }
                ]
            }
        }
    ]
}
```

**Coverage:**
```json
{
    "resourceType": "Coverage",
    "id": "123456",
    "status": "active",
    "beneficiary": {
        "reference": "Patient/12345"
    },
    "payor": [
        {
            "reference": "Organization/Aetna"
        }
    ]
}
