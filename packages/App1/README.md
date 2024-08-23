This app is not meant to be run. This is only a testing repo for a react-native issue.

App1 declares an old version (3.2.1) of @react-native-community/geolocation

This version **SHOULD** cause codegen to issue a deprecation warning.


# NOTE
The bug being illustrated is **NOT** about the geolocation dependency. The dep is just being used as a known trigger of the `codegen` deprecation warning which is an indicator of the problematic dep resolving process in codegen itself. Any Dep that has >= 2 versions and has fixed its package.json in one of them could be used to demonstrate the issue in question.
