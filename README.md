# Python, paho, mqtt and AWS IoT

## Platforms supported

I've tested the certificate creation commands only on Windows using the AWS CLI. I think they should work on the AWS CLI of other platforms.

My python programs run perfectly on:
- Raspberry PI 2 with Raspbian Jessie and Python 2.7
- Debian Jessie virtual machine with Python 2.7
- Windows with Python 3.4 installed by Conda

## Create a thing, certifcate, keys and attaching them to enable usage of AWS IoT hub

Based on AWS docs found here: [http://docs.aws.amazon.com/iot/latest/developerguide/secure-communication.html](http://docs.aws.amazon.com/iot/latest/developerguide/secure-communication.html)

Create one thing in aws IoT:
```
aws iot create-thing --thing-name "myThingName"
```
list the things you now have:
```
aws iot list-things
```
create certificate and keys and set as active:
```
aws iot create-keys-and-certificate --set-as-active --certificate-pem-outfile cert.pem --public-key-outfile publicKey.pem --private-key-outfile privkey.pem
```
take note of the **certificate-arn** in the output or, if you forgot to copy the **certificate-arn** you can get it listing the certificates with:
```
aws iot list-certificates
```
download root certificate from [this URL](https://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem) using your browser and save it with filename: **aws-iot-rootCA.crt**

KD: Check IoT policies as this existing in my account from earlier tutorials
create a policy from the file provided:
```
aws iot create-policy --policy-name "PubSubToAnyTopic" --policy-document file://iotpolicy.json
```
KD: associate the newly created certificate with the policy created in the previous step to permit publishing to any topic name
paste your **certificate-arn** inside the following command before entering it:
```
aws iot attach-principal-policy --principal "certificate-arn" --policy-name "PubSubToAnyTopic"
```
KD: You can also get the endpoint from the AWS Console -> IoT Dashboard as this is account/region specific
Two options about the configuration of your endpoint:
- change the value of awshost using the returned value of "endpointAddress":
```
aws iot describe-endpoint
```
- use **data** hostname and specify the region with the one you used to create the thing and certificates. Current sample code contains data.iot.**eu-west-1**.amazonaws.com   (I will try to understand which are the benefits, if any, of one over the other).

KD: Now attach the certificate to the Thing you created earlier.  The certificate is what associates the Policy/Thing combo
At this point my sample python programs ( **awsiotpub.py**  and  **awsiotsub.py** ) should run correctly but the AWS documentation specifies to also enter the following to attach the certificate to the thing:

```
aws iot attach-thing-principal --thing-name "myThingName" --principal "certificate-arn"
```
## How to test the sample Python programs

- open two console windows and enter in the first **awsiotsub.py** and in the second **awsiotpub.py**
- the second one will start sending random temperature values to the AWS IoT hub
- the first one will display them when received from the IoT hub

You can check the sources and modify the topics used by both programs to better fit your needs.
Currently, **awsiotsub.py** subscribes to any topic and will show all of the received msgs.

KD: Both python scripts will need to be updated.  At the very least, need the correct AWS IoT Hub arn
KD: If you customized the cert, key or thing name earlier in the tutorial then make sure to update scripts accordingly.

**Enjoy MQTT and AWS IoT in your Python programs!**
