# 封禁分析

## 环境
> 测试环境上线WAF后，get到一些访问被误杀，分析其原因。

## 误杀

``` yaml
``` yaml
{
  "message": "URL Encoding Abuse Attack Attempt",
  "details": {
    "reference": "o0,33v129,33o1149,1o1075,1o1047,1o863,1o681,1o377,1v373,1715o377,1715v373,1715",
    "ver": "OWASP_CRS/3.2.0",
    "file": "/data/openresty/nginx/modsec/owasp-modsecurity-crs-3.2.0/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf",
    "data": "application/x-www-form-urlencoded",
    "lineNumber": "347",
    "ruleId": "920240",
    "match": "Matched \"Operator `ValidateUrlEncoding' with parameter `' against variable `REQUEST_BODY' (Value: `\\x1f\\x8b\\x08\\x00\\x00\\x00\\x00\\x00\\x00\\x13\\xed[\\xdbn\\xdbF\\x10\\xfd\\x95\\x82O-`\\xb3{\\xbf\\x08\\xe8\\x83\\xe2$ (4834 characters omitted)' )",
    "accuracy": "0",
    "rev": "",
    "maturity": "0",
    "severity": "4",
    "tags": [
      "application-multi",
      "language-multi",
      "platform-multi",
      "attack-protocol",
      "OWASP_CRS",
      "OWASP_CRS/PROTOCOL_VIOLATION/EVASION"
    ]
  }
},
{
  "message": "Invalid character in request (null character)",
  "details": {
    "reference": "o281,1v652,399t:urlDecodeUnio236,1o303,1o366,1o405,1v1578,478t:urlDecodeUnio5,1o6,1v2081,7t:urlDecodeUnio3,1o4,1o5,1o6,1o7,1o8,1v373,36t:urlDecodeUnio6,1v1161,60t:urlDecodeUnio165,1v1257,220t:urlDecodeUni",
    "ver": "OWASP_CRS/3.2.0",
    "file": "/data/openresty/nginx/modsec/owasp-modsecurity-crs-3.2.0/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf",
    "data": "ARGS_NAMES:�F��Iq�\u0005��.Lӹ���<���\u0007i�����_\u001e����އ��y���\u000f��x���;���bH�\rH\u0016� �\rArQ|���\f\u0017\u001d��5��b$�\u0002#�Q5\f�ob$�\u001c\u000br2��f�)#\fo�\u0003��p�<���4�ȅӸ\u0006$��F���\u001a!J�;�$�\u000e$w\u0001�ER\u0002��\u0016i���#�\u0006�Twi�|%�",
    "ruleId": "920270",
    "lineNumber": "467",
    "match": "Matched \"Operator `ValidateByteRange' with parameter `1-255' against variable `ARGS_NAMES:�F��Iq�\u0005��.Lӹ���<���\u0007i�����_\u001e����އ��y���\u000f��x���;���bH�\rH\u0016� �\rArQ|���\f\u0017\u001d��5��b$�\u0002#�Q5\f�ob$�\u001c\u000br2��f�)#\fo�\u0003��p�<���4�ȅӸ\u0006$��F���\u001a!J�;�$�\u000e$w\u0001�ER\u0002��\u0016i���#�\u0006�Twi�|%�",
    "accuracy": "0",
    "rev": "",
    "maturity": "0",
    "severity": "2",
    "tags": [
      "application-multi",
      "language-multi",
      "platform-multi",
      "attack-protocol",
      "OWASP_CRS",
      "OWASP_CRS/PROTOCOL_VIOLATION/EVASION"
    ]
  }
},
{
  "message": "HTTP Header Injection Attack via payload (CR/LF detected)",
  "details": {
    "reference": "o3,1v1052,51t:urlDecodeUni,t:htmlEntityDecodeo15,1v1104,46t:urlDecodeUni,t:htmlEntityDecodeo148,1v410,152t:urlDecodeUni,t:htmlEntityDecodeo53,1v1161,60t:urlDecodeUni,t:htmlEntityDecodeo59,1o65,1v1257,220t:urlDecodeUni,t:htmlEntityDecodeo62,1o96,1v1478,99t:urlDecodeUni,t:htmlEntityDecode",
    "ver": "OWASP_CRS/3.2.0",
    "file": "/data/openresty/nginx/modsec/owasp-modsecurity-crs-3.2.0/rules/REQUEST-921-PROTOCOL-ATTACK.conf",
    "data": "Matched Data: \n found within ARGS_NAMES: �\\�u'�Wh�X4B�E���W�F�\u0003�[����\u0003�7,]b�~�6{��\u0003(%�(m\u0014\u000b�eޓW��*��U\u001e\n-�ai���b��4Uw#\u0003��\\\u001f-\u001a�\u00131���`��|��\n\u000ff:  �\\�u'�Wh�X4B�E���W�F�\u0003�[����\u0003�7,]b�~�6{��\u0003(%�(m\u0014\u000b�eޓW��*��U\u001e\n-�ai���b��4Uw#\u0003��\\\u001f-\u001a�\u00131���`��|��\n\u000ff",
    "ruleId": "921150",
    "lineNumber": "148",
    "match": "Matched \"Operator `Rx' with parameter `[\\n\\r]' against variable `ARGS_NAMES: �\\�u'�Wh�X4B�E���W�F�\u0003�[����\u0003�7,]b�~�6{��\u0003(%�(m\u0014\u000b�eޓW��*��U\u001e\n-�ai���b��4Uw#\u0003��\\\u001f-\u001a�\u00131���`��|��\n\u000ff' (Value: ` \\x8d\\\\x82u'\\xd0Wh\\xc7X4B\\xbeE\\xe9\\xb6\\xc8W\\x8eF\\xac\\x03\\xe0[\\xb4\\xac\\xa3\\xe2\\x03\\x9c7,]b\\xab~\\xcb6{ (170 characters omitted)' )",
    "accuracy": "0",
    "rev": "",
    "maturity": "0",
    "severity": "2",
    "tags": [
      "application-multi",
      "language-multi",
      "platform-multi",
      "attack-protocol",
      "OWASP_CRS",
      "OWASP_CRS/WEB_ATTACK/HEADER_INJECTION"
    ]
  }
},
{
  "message": "Inbound Anomaly Score Exceeded (Total Score: 57)",
  "details": {
    "reference": "",
    "ver": "",
    "file": "/data/openresty/nginx/modsec/owasp-modsecurity-crs-3.2.0/rules/REQUEST-949-BLOCKING-EVALUATION.conf",
    "data": "",
    "lineNumber": "79",
    "ruleId": "949110",
    "match": "Matched \"Operator `Ge' with parameter `5' against variable `TX:ANOMALY_SCORE' (Value: `57' )",
    "accuracy": "0",
    "rev": "",
    "maturity": "0",
    "severity": "2",
    "tags": [
      "application-multi",
      "language-multi",
      "platform-multi",
      "attack-generic"
    ]
  }
},
{
  "message": "Inbound Anomaly Score Exceeded (Total Inbound Score: 57 - SQLI=0,XSS=0,RFI=0,LFI=0,RCE=0,PHPI=0,HTTP=30,SESS=0): individual paranoia level scores: 57, 0, 0, 0",
  "details": {
    "reference": "",
    "file": "/data/openresty/nginx/modsec/owasp-modsecurity-crs-3.2.0/rules/RESPONSE-980-CORRELATION.conf",
    "ver": "",
    "data": "",
    "lineNumber": "76",
    "ruleId": "980130",
    "match": "Matched \"Operator `Ge' with parameter `5' against variable `TX:INBOUND_ANOMALY_SCORE' (Value: `57' )",
    "accuracy": "0",
    "rev": "",
    "maturity": "0",
    "severity": "0",
    "tags": [
      "event-correlation"
    ]
  }
}
```


```