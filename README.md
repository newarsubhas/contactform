# verify email in aws ses 
# create lambda function in nodejs 6.10
# index.js
```
console.log('Loading function');

var AWS = require('aws-sdk');
var doc = require('dynamodb-doc');
var dynamo = new AWS.DynamoDB({params: {TableName: 'contactformenopits'}}); // edit tame nane
var ses = new AWS.SES();

exports.handler = function(event, context) {
    //console.log('Received event:', JSON.stringify(event, null, 2)); //DEBUG
    // Get values from event and assign to variables.
    var DateTime = new Date().toString();
    var Name = event.name;
    var Email = event.email;
    var PhoneNumber = event.phonenumber;
    var Subject = event.subject;
    var Message = event.message;
    var emailSubject;
    var emailBody;
    var emailParams;

    // Message is the only required field
    // If it isn't empty create the itemParams json
    // putItem in DynamoDB
    // send via SES
    if(Message.length > 0) {
      var item = {
        'datetime': {'S': DateTime},
        'message': {'S': Message}
      };
      // Name, Email, and Subject are optional, only add them to the
      // item object if they are submitted.
      if(Name) item.name = {'S': Name};
      if(Email) item.email = {'S': Email};
      if(PhoneNumber) item.phonenumber = {'S': PhoneNumber};
      if(Subject) item.subject = {'S': Subject};
      // Put the item object in the itemParams object.
      var itemParams = {
        Item: item
      };
      dynamo.putItem(itemParams, function(err, data){
        if (err) {
              //console.log('Error putting item in dynamodb: '+err); //DEBUG
              context.fail(err);
            } else {
              //DynamoDB put successfull, send the email.
              //console.log('DynamoDB put successful'); //DEBUG
              emailSubject = "Cuntact form from enopits";
              emailBody = "Date Submitted: "+DateTime+"<br/>\r\nTheir Name: ";
              if(Name){ emailBody+=Name+"<br/>\r\nEmail Address: ";}
                else{ emailBody+="Not given<br/>\r\nEmail Address: ";}
              if(Email){ emailBody+=Email+"<br/>\r\nPhoneNumber: ";}
                else{ emailBody+="Not given<br/>\r\nPhoneNumber: ";}
              if(PhoneNumber){ emailBody+=PhoneNumber+"<br/>\r\nSubject: ";}
                else{ emailBody+="Not given<br/>\r\nSubject: ";}
              if(Subject){ emailBody+=Subject+"<br/>\r\n";}
                else{ emailBody+="Not given<br/>\r\n";}
              emailBody+="Message: "+Message+"<br/>\r\n";
              emailParams = {
                Destination: {
                      ToAddresses: ["prabhat@enopits.com"]   // your verifies ses email 
                },
                Message: {
                    Body: {
                    Html: {
                        Data: emailBody,
                        Charset: 'UTF-8'
                    }
                  },
                  Subject: {
                    Data: emailSubject,
                    Charset: 'UTF-8'
                  }
                },
                ReplyToAddresses: ["prabhat@enopits.com"],  // your verifies ses email 
                ReturnPath: "prabhat@enopits.com",  // your verifies ses email 
                Source: "prabhat@enopits.com"  // your verifies ses email 
              };
              ses.sendEmail(emailParams,function(err, data) {
                if (err) {
                  //console.log("Email did not send."); //DEBUG
                  context.fail('Error sending email:'+err+err.stack); //an error occurred with SES
                } else {
                  //console.log("Email sent."+data);  //DEBUG SES send successful
                  context.succeed("true");
                }
              });
            }
          });
    } else {
      context.fail("Message is a required field.");
    }
};
```
# create aws api gateway 
and use this lambda function .

update https://github.com/prabhatpankaj/contactform/blob/master/js/contact_me.js

url: "https://<yourAPI>.execute-api.us-west-2.amazonaws.com/prod/contactform",
