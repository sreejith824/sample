## User
```mermaid
sequenceDiagram

Actor  user as User
participant form as Verge Registration form
participant formapi as Verge Form Submission Exp API
participant docapi as Document exchange API
participant center as Contact Center API
participant gen as Genesys
participant genws as Genesys workspace
participant rpa as Robotic process

user ->> form : fill up the form and upload document(s)
form ->> formapi : REST API for form submission
activate formapi

formapi ->> docapi : Upload documnet(s)
activate docapi
docapi ->> docapi : Store documents temporarily (eg. S3)
docapi -->> formapi : return URL for Document Viewer app + querystring as context   
deactivate docapi

formapi ->> formapi :  Assemble input for contact center, including document urls 
formapi ->> center : Submit form (REST API)
activate center
center ->> gen : Webform JSON structure
activate gen
gen ->> genws : form data
activate genws
deactivate genws
gen ->> rpa : Webform JSON structure
activate rpa
deactivate rpa

deactivate gen    
center -->> formapi :    
deactivate center            
formapi -->> form: 
deactivate formapi
form -->> user : Confirmation message     
```

- Customer upload documents in the Webform, but keep the document in the browser.
- On form submission, captured data and uploaded document are submitted to API.
- From API, upload document to S3 through Document Exchange API. In return get <b>application URL to access the document </b>
- Add <b>application URL to access the document </b> to Webform payload. An example as below,.

```json
{
    "webformName": "Verge form",
    "webformVersion": "1",
    "timestamp": "2022-09-30T12:18:44.668Z",
    "routingTag": "PMO:Kontoapne:Verge:Rollerogtilganger",
    "from": "",
    "to": "no-reply@dnb.no",
    "subject": "Verge form",
    "content": [
        {
            "label": "Oppsummering",
            "key": "SUMMARY_HEADER",
            "value": "",
            "contentFormat": "HDR"
        },
        {
            "label": "List of Accounts to change :",
            "value": "",
            "key": "CHANGE_ACCOUNTS_LIST",
            "contentFormat": "LIST",
            "listValues": [
                {
                    "value": [
                        {
                            "label": "Kontodetaljer for konto nr.:",
                            "value": "1",
                            "key": "CHANGE_ACCOUNT_COUNT"
                        },
                        {
                            "label": "Kontonummer",
                            "key": "CHANGE_ACCOUNT_NUMBER",
                            "value": "12266706780"
                        }

                    ]
                },
                {
                    "value": [
                        {
                            "label": "Kontodetaljer for konto nr.:",
                            "value": "2",
                            "key": "CHANGE_ACCOUNT_COUNT"
                        },
                        {
                            "label": "Kontonummer",
                            "key": "CHANGE_ACCOUNT_NUMBER",
                            "value": "12266702351"
                        }

                    ]
                }
            ]
        }
    ],
    "lookupItems": [
        {
          "lookupId": "Endpoint with querystring to get document for this context",
          "querystring": "",
          "tag": ""
        }
    ]
}
```

## Advisor

```mermaid
sequenceDiagram

Actor advisor as Advisor
participant genws as Genesys workspace

participant docapp as Document exchange Web App
participant ad as Active Directory
participant docarc as Doc Ark

advisor ->> genws : Work on request
activate genws
genws --> genws : open case
genws ->> docapp : URL for Document Viewer app + querystring as context 
deactivate genws

activate docapp
docapp ->> ad : SSO
docapp ->> docapp : Get Document(s) for context from temporary storage
deactivate docapp


advisor ->> docapp : Review documents
activate docapp
deactivate docapp

advisor ->> docapp : Archive document(s)
activate docapp
docapp ->> docarc : Archive documents(s)
activate docarc
docarc -->> docapp : return
deactivate docarc

deactivate docapp
```

- Advisor open the case in Genesys Workspace
- Click link <b> application URL to access the document </b>
- A browser open ups to  redirect to Document exchange web app
- Web app redirect to Azure AD for SSO
- Once SSO, web app get documents from S3.
- Advisor previews documents
- Archive the document to Doc Ark


### Timeline
- 
