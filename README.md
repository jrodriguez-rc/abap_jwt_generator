![abapLint](https://github.com/abapChaoLiu/abap_jwt_generator/workflows/abapLint/badge.svg?branch=master)

# ABAP JWT Generator

## Simple Demo
```abap
REPORT zrpt_jwt_generator_demo.

DATA: jwt_generator TYPE REF TO zcl_jwt_generator.

DATA: jwt_header TYPE zcl_jwt_generator=>ty_jwt_header,
      jwt_claim  TYPE zcl_jwt_generator=>ty_jwt_claim.

DATA: current_timestamp TYPE timestamp,
      exp_timestamp     TYPE tzntstmpl,
      diff_second       TYPE tzntstmpl,
      exp_second        TYPE int8,
      ssfinfo           TYPE ssfinfo.

DATA: start_timestamp TYPE timestamp VALUE '19700101000000',
      ssf_id          TYPE ssfid VALUE '<implicit>',
      "PSE profile with private key
      ssf_profile     TYPE ssfprof VALUE 'SAPZJWTSF001.pse'.


jwt_header-alg  = 'RS256'.

GET TIME STAMP FIELD current_timestamp.
cl_abap_tstmp=>add(
  EXPORTING
    tstmp                      =  current_timestamp   " UTC Time Stamp
    secs                       =  120   " Time Interval in Seconds
  RECEIVING
    r_tstmp                    =  exp_timestamp   ). " UTC Time Stamp
cl_abap_tstmp=>subtract(
  EXPORTING
    tstmp1                     =  exp_timestamp   " UTC Time Stamp
    tstmp2                     =  start_timestamp   " UTC Time Stamp
  RECEIVING
    r_secs                     =  diff_second  ). " Time Interval in Seconds

exp_second = diff_second.
jwt_claim = VALUE #(
  iss = 'UserID'
  sub = 'example@gmail.com'
  aud = 'https://login.example.com'
  exp = exp_second ).


ssfinfo = VALUE #( id = ssf_id profile = ssf_profile ).

CREATE OBJECT jwt_generator.
TRY.
    jwt_generator->generate_jwt(
      EXPORTING
        jwt_header     = jwt_header
        jwt_claim      = jwt_claim
        ssf_info       = ssfinfo
      RECEIVING
        jwt            = DATA(jwt) ).
  CATCH zcx_jwt_generator INTO DATA(lo_exp).
    WRITE /: 'ERROR when generate JWT token.'.
    WRITE /: lo_exp->get_text( ).
ENDTRY.
write: lv_where_clause.
```

## Credits and references

Class ZCL_JWT_Generator is modified from Dimitri Seifmann's post [Connect from AS ABAP to Google Cloud Platform App-Engine resource secured with Google Identity-Aware Proxy](https://blogs.sap.com/2019/11/10/connect-from-as-abap-to-google-cloud-platform-app-engine-resource-secured-with-google-identity-aware-proxy/).
