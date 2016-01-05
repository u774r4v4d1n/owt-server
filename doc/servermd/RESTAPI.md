WebRTC RESTAPI
-------
# 1 Introduction {#section1}
Intel WebRTC solution provides a set of REST APIs for conference management. Manager clients can be implemented by different programming languages.

# 2 Authentication and Authorization {#section2}
Each request should be a HTTP request with "Authorization" field in the header. So the server can determine whether it is a valid client. The value of "Authorization" is something like
MAuth　realm=http://webrtc.intel.com,mauth_signature_method=HMAC_SHA1,mauth_username=test,mauth_role=role,mauth_serviceid=53c74879209ee7f96e5cbc9c,mauth_cnonce=87428,mauth_timestamp=1406079112038,mauth_signature=ZjA5NTJlMjE0ZTY4NzhmZWRjZDkxYjNmZjkyOTIxYzMyZjg3NDBjZA==
MAuth realm=http://webrtc.intel.com,mauth_signature_method=HMAC_SHA1 is a constant string which should not be changed.

 - mauth_username and mauth_role are optional. Some request may require username and role. Then it should also be included in the header.
 - mauth_serviceid is the ID of the service.
 - mauth_cnonce is a random value between 0 and 99999.
 - mauth_timestamp is the timestamp of the request.
 - mauth_signature is signature of this request. It uses sha1 to sign timestamp, cnonce,
 - username(optional), role(optional) with service key. Then encrypted it with BASE64.

Example of encryption algorithm (python):　
~~~~~~~~~~~~~{.py}
toSign = str(timestamp) + ',' + str(cnounce);
if (username != '' and role != ''):
  toSign += ',' + username + ',' + role;
signed = self.calculateSignature(toSign, key);
def calculateSignature(self, toSign, key):
  from hashlib import sha1
  import hmac
  import binascii
  hash = hmac.new(key, toSign, sha1);
  signed = binascii.b2a_base64(hash.hexdigest())[:-1];
  return signed;
~~~~~~~~~~~~~

toSign is the signature.

# 3 APIs {#section3}

## 3.1 Get Rooms {#section3_1}

This API lists all the rooms that available in current service.

**URL**

        http://<hostname>:3000/rooms

**HTTP Method**

        GET

**Response Example**

~~~~~~~~~~~~~{.js}
[{'limit_in': 16,
  'name': 'Mix Room',
  'codec': None,
  '_id': '53c748b9f59ad28f4a3e54dc',
  'quality': 'Unspecified'},
 {'limit_in': 16,
  'name': 'Forward Room',
  'codec': None,
  '_id': '53c748b9f59ad28f4a3e54dd',
  'quality': 'Unspecified'}]
~~~~~~~~~~~~~

## 3.2 Create Room {#section3_2}

This API creates a room in current service.

**URL**

        http://<hostname>:3000/rooms/

**HTTP Method**

        POST

**Parameters**

 - **Option**: A JSON string.  Following attributes are allowed: limit_in, limit_all, quality, mode, codec.

**Response Example**

~~~~~~~~~~~~~{.js}
{'limit_in': 16,
 name: 'test room',
'mode': 'mix',
'_id': '53cdc8d6232fd73f0bf6f966',
'quality': 'Unspecified'}
~~~~~~~~~~~~~

## 3.3 Create Token {#section3_3}

This API creates a token for given user. Client can connect to specified room by this token.

**URL**

        http://<hostname>:3000/rooms/[room id]/tokens

**HTTP Method**

        POST

**Response Example**

~~~~~~~~~~~~~{.js}
eyJ0b2tlbklkIjoiNTNjZGNkMWU1Y2UzMTkxMjIyOWFjYTRjIiwiaG9zdCI6IjE5Mi4xNjguNDIuODg6ODA4MCIsInNpZ25hdHVyZSI6Ik1qQmhOVFZtWTJKa1pqZGhOMkk0T0RGa1lqTXhObUV5T1dVNE9USmtPVGt3TmpkbVpHRXdNUT09In0=
~~~~~~~~~~~~~

**Remarks**

The token returned is encrypted with base 64. Client should decrypt it to get a JSON object. It has a property named host which is the hostname of the worker.

## 3.4 Get User List {#section3_4}

This API lists all the users who joined the room of room ID.

**URL**

        http://<hostname>:3000/rooms/[room id]/users

**HTTP Method**

        GET

**Response Example**

~~~~~~~~~~~~~{.js}
[{'role': 'role',
  'name': 'user'},
 {'role': 'role',
 'name': 'user'}]
~~~~~~~~~~~~~