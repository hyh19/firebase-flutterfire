# Profile.dart Multi-Factor Authentication Features

## Multi-Factor Authentication Overview

This file demonstrates the complete implementation of Firebase Auth Multi-Factor Authentication (MFA), including phone number verification and TOTP (Time-based One-Time Password) authentication methods.

## Get Enrolled MFA Factors

```dart 185:191:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      TextButton(
                        onPressed: () async {
                          final a = await user.multiFactor.getEnrolledFactors();
                          print(a);
                        },
                        child: const Text('Get enrolled factors'),
                      ),
```

Retrieves and prints all MFA factors enrolled by the user:

- Calls `user.multiFactor.getEnrolledFactors()` to get enrolled factor list
- Prints results for debugging and viewing current MFA status

## Phone Number Multi-Factor Authentication

### Phone Number Input Component

```dart 211:218:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      TextFormField(
                        controller: phoneController,
                        decoration: const InputDecoration(
                          icon: Icon(Icons.phone),
                          hintText: '+33612345678',
                          labelText: 'Phone number',
                        ),
                      ),
```

Phone number input field:

- Uses TextFormField component
- Includes phone icon
- Hint shows example format: +33612345678
- Binds input value through phoneController

### Phone MFA Enrollment Flow

```dart 220:256:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      TextButton(
                        onPressed: () async {
                          final session = await user.multiFactor.getSession();
                          await auth.verifyPhoneNumber(
                            multiFactorSession: session,
                            phoneNumber: phoneController.text,
                            verificationCompleted: (_) {},
                            verificationFailed: print,
                            codeSent: (
                              String verificationId,
                              int? resendToken,
                            ) async {
                              final smsCode = await getSmsCodeFromUser(context);

                              if (smsCode != null) {
                                // Create a PhoneAuthCredential with the code
                                final credential = PhoneAuthProvider.credential(
                                  verificationId: verificationId,
                                  smsCode: smsCode,
                                );

                                try {
                                  await user.multiFactor.enroll(
                                    PhoneMultiFactorGenerator.getAssertion(
                                      credential,
                                    ),
                                  );
                                } on FirebaseAuthException catch (e) {
                                  print(e.message);
                                }
                              }
                            },
                            codeAutoRetrievalTimeout: print,
                          );
                        },
                        child: const Text('Verify Number For MFA'),
                      ),
```

Phone MFA enrollment flow:

1. **Get session**: `user.multiFactor.getSession()` - Creates MFA session
2. **Send verification code**: `auth.verifyPhoneNumber()` - Sends SMS verification code to specified phone number
3. **User input verification code**: `getSmsCodeFromUser(context)` - Shows dialog for user to input received verification code
4. **Create credential**: `PhoneAuthProvider.credential()` - Creates credential using verification code and verification ID
5. **Enroll MFA factor**: `user.multiFactor.enroll()` - Registers phone number as MFA factor
6. **Error handling**: Catches FirebaseAuthException and prints error message

## TOTP Multi-Factor Authentication

### TOTP Enrollment Flow

```dart 257:296:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      TextButton(
                        onPressed: () async {
                          final totp =
                              (await user.multiFactor.getEnrolledFactors())
                                  .firstWhereOrNull(
                            (element) => element.factorId == 'totp',
                          );
                          if (totp != null) {
                            await user.multiFactor.unenroll(
                              factorUid:
                                  (await user.multiFactor.getEnrolledFactors())
                                      .firstWhere(
                                        (element) => element.factorId == 'totp',
                                      )
                                      .uid,
                            );
                          }
                          final session = await user.multiFactor.getSession();
                          final totpSecret =
                              await TotpMultiFactorGenerator.generateSecret(
                            session,
                          );
                          print(totpSecret);
                          final code =
                              await getTotpFromUser(context, totpSecret);
                          print('code: $code');
                          if (code == null) {
                            return;
                          }
                          await user.multiFactor.enroll(
                            await TotpMultiFactorGenerator
                                .getAssertionForEnrollment(
                              totpSecret,
                              code,
                            ),
                            displayName: 'TOTP',
                          );
                        },
                        child: const Text('Enroll TOTP'),
                      ),
```

TOTP enrollment flow:

1. **Check existing TOTP**: If TOTP factor already exists, unenroll it first
2. **Get session**: Create new MFA session
3. **Generate secret**: `TotpMultiFactorGenerator.generateSecret()` - Generates TOTP secret key
4. **Display secret**: Prints secret information (user needs to enter this in authenticator app)
5. **Get verification code**: `getTotpFromUser(context, totpSecret)` - Prompts user to input code generated by authenticator
6. **Enroll TOTP factor**: Completes registration using secret key and verification code
7. **Set display name**: Sets display name "TOTP" for the TOTP factor

### TOTP Unenrollment Logic

The TOTP enrollment flow includes logic to automatically unenroll existing TOTP factors:

```dart 259:274:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                          final totp =
                              (await user.multiFactor.getEnrolledFactors())
                                  .firstWhereOrNull(
                            (element) => element.factorId == 'totp',
                          );
                          if (totp != null) {
                            await user.multiFactor.unenroll(
                              factorUid:
                                  (await user.multiFactor.getEnrolledFactors())
                                      .firstWhere(
                                        (element) => element.factorId == 'totp',
                                      )
                                      .uid,
                            );
                          }
```

If an existing TOTP factor is found, it calls `user.multiFactor.unenroll()` to unenroll it before registering the new TOTP factor.

## MFA Unenrollment Features

```dart 297:313:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      TextButton(
                        onPressed: () async {
                          try {
                            final enrolledFactors =
                                await user.multiFactor.getEnrolledFactors();

                            await user.multiFactor.unenroll(
                              factorUid: enrolledFactors.first.uid,
                            );
                            // Show snackbar
                            ScaffoldSnackbar.of(context).show('MFA unenrolled');
                          } catch (e) {
                            print(e);
                          }
                        },
                        child: const Text('Unenroll MFA'),
                      ),
```

Universal MFA unenrollment features:

1. **Get enrolled factors**: Retrieves all enrolled MFA factors
2. **Unenroll first factor**: Uses `unenroll()` method to unenroll the first factor
3. **Success message**: Shows "MFA unenrolled" message
4. **Error handling**: Catches exceptions and prints error information

## Helper Functions Explanation

The code uses two helper functions (defined in auth.dart):

### getSmsCodeFromUser(context)

Dialog function for getting SMS verification code input from user.

### getTotpFromUser(context, totpSecret)

Dialog function for getting TOTP verification code input from user, receives TOTP secret parameter for displaying secret information.

## Multi-Factor Authentication Flow Summary

Firebase Auth multi-factor authentication implementation includes the following key steps:

1. **Session management**: Uses `getSession()` to create MFA sessions
2. **Factor enrollment**: Uses `enroll()` method to register new authentication factors
3. **Factor unenrollment**: Uses `unenroll()` method to remove authentication factors
4. **Credential verification**: For phone numbers, uses SMS verification codes; for TOTP, uses time-based verification codes
5. **Error handling**: Catches FirebaseAuthException for appropriate error handling

This implementation demonstrates complete MFA lifecycle management, including enrollment, verification, and unenrollment of all core features.
