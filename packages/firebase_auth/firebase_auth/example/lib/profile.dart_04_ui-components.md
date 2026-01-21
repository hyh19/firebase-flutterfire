# Profile.dart UI Components and Interface Logic

## Main UI Build Method

### Build Method Overview

```dart 97:103:packages/firebase_auth/firebase_auth/example/lib/profile.dart
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: FocusScope.of(context).unfocus,
      child: Scaffold(
        body: Stack(
          children: [
```

Main UI structure:

- **GestureDetector**: Dismisses keyboard when tapping blank areas
- **Scaffold**: Provides basic Material Design layout structure
- **Stack**: Uses stacked layout to implement floating save button positioning

## Page Layout Structure

### Main Content Area

```dart 103:106:packages/firebase_auth/firebase_auth/example/lib/profile.dart
            Center(
              child: SizedBox(
                width: 400,
                child: SingleChildScrollView(
                  child: Column(
```

Main layout characteristics:

- **Center alignment**: Uses Center component for content centering
- **Fixed width**: SizedBox limits maximum width to 400 pixels for good mobile experience
- **Scroll view**: SingleChildScrollView supports content scrolling
- **Vertical layout**: Column arranges all components vertically

### Main Component Arrangement

```dart 107:109:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: [
                      Stack(
                        children: [
```

All components are contained within a Column, arranged vertically and centered.

## Avatar Component Details

### Avatar Display Area

```dart 110:144:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      Stack(
                        children: [
                          CircleAvatar(
                            maxRadius: 60,
                            backgroundImage: NetworkImage(
                              user.photoURL ?? placeholderImage,
                            ),
                          ),
                          Positioned.directional(
                            textDirection: Directionality.of(context),
                            end: 0,
                            bottom: 0,
                            child: Material(
                              clipBehavior: Clip.antiAlias,
                              color: Theme.of(context).colorScheme.secondary,
                              borderRadius: BorderRadius.circular(40),
                              child: InkWell(
                                onTap: () async {
                                  final photoURL = await getPhotoURLFromUser();

                                  if (photoURL != null) {
                                    await user.updatePhotoURL(photoURL);
                                  }
                                },
                                radius: 50,
                                child: const SizedBox(
                                  width: 35,
                                  height: 35,
                                  child: Icon(Icons.edit),
                                ),
                              ),
                            ),
                          ),
                        ],
                      ),
```

Avatar component design:

1. **Circular avatar**: CircleAvatar displays user avatar or default placeholder
2. **Edit button**: Floating circular edit button in bottom-right corner
3. **Material design**: Uses Material component for touch feedback
4. **Responsive layout**: Uses Positioned.directional to adapt to different text directions

## Text Input Components

### Display Name Input Field

```dart 145:159:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      const SizedBox(height: 10),
                      TextField(
                        textAlign: TextAlign.center,
                        controller: controller,
                        decoration: const InputDecoration(
                          border: InputBorder.none,
                          floatingLabelBehavior: FloatingLabelBehavior.never,
                          alignLabelWithHint: true,
                          label: Center(
                            child: Text(
                              'Click to add a display name',
                            ),
                          ),
                        ),
                      ),
```

Display name input field characteristics:

- **Center alignment**: Text displayed centered
- **Borderless design**: Maintains consistent page styling
- **Placeholder hint**: "Click to add a display name"
- **Centered label**: Label text also centered

### User Information Display

```dart 160:161:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      Text(user.email ?? user.phoneNumber ?? 'User'),
```

User information display logic:

- Shows email address first priority
- Shows phone number if no email available
- Shows default text "User" if neither exists

## Sign-in Provider Icon Display

```dart 162:177:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      Row(
                        mainAxisAlignment: MainAxisAlignment.center,
                        children: [
                          if (userProviders.contains('phone'))
                            const Icon(Icons.phone),
                          if (userProviders.contains('password'))
                            const Icon(Icons.mail),
                          if (userProviders.contains('google.com'))
                            SizedBox(
                              width: 24,
                              child: Image.network(
                                'https://upload.wikimedia.org/wikipedia/commons/0/09/IOS_Google_icon.png',
                              ),
                            ),
                        ],
                      ),
```

Sign-in method icon display:

- **Conditional rendering**: Shows corresponding icons based on user's sign-in providers
- **Icon selection**:
  - Phone sign-in: Phone icon
  - Email/password sign-in: Mail icon
  - Google sign-in: Google logo from web image
- **Layout**: Horizontal arrangement, center aligned

## Function Button Area

### Email Verification Button

```dart 178:184:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      const SizedBox(height: 20),
                      TextButton(
                        onPressed: () {
                          user.sendEmailVerification();
                        },
                        child: const Text('Verify Email'),
                      ),
```

### Get Enrolled Factors Button

```dart 185:191:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      TextButton(
                        onPressed: () async {
                          final a = await user.multiFactor.getEnrolledFactors();
                          print(a);
                        },
                        child: const Text('Get enrolled factors'),
                      ),
```

### Apple Token Revocation Button

```dart 192:210:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      TextButton(
                        onPressed: () async {
                          if (AuthGate.appleAuthorizationCode != null) {
                            await FirebaseAuth.instance
                                .revokeTokenWithAuthorizationCode(
                              AuthGate.appleAuthorizationCode!,
                            );
                            AuthGate.appleAuthorizationCode = null;
                          } else {
                            print(
                              'Apple `authorizationCode` is null, cannot revoke token.',
                            );
                          }
                        },
                        child: const Text('Revoke Apple auth token'),
                      ),
```

## Multi-Factor Authentication UI Components

### Phone Number Input Field

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

### Phone MFA Enrollment Button

```dart 219:256:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      const SizedBox(height: 20),
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

### TOTP Enrollment Button

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

### MFA Unenrollment Button

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

### Sign Out Button

```dart 314:318:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      const Divider(),
                      TextButton(
                        onPressed: _signOut,
                        child: const Text('Sign out'),
                      ),
```

## Floating Save Button

### Save Button Component

```dart 324:337:packages/firebase_auth/firebase_auth/example/lib/profile.dart
            Positioned.directional(
              textDirection: Directionality.of(context),
              end: 40,
              top: 40,
              child: AnimatedSwitcher(
                duration: const Duration(milliseconds: 200),
                child: !showSaveButton
                    ? SizedBox(key: UniqueKey())
                    : TextButton(
                        onPressed: isLoading ? null : updateDisplayName,
                        child: const Text('Save changes'),
                      ),
              ),
            ),
```

Floating save button characteristics:

- **Positioning**: Top-right corner (end: 40, top: 40)
- **Animated switching**: AnimatedSwitcher for smooth show/hide transitions
- **Conditional display**: Only shows when display name has changed
- **State control**: Button disabled during loading
- **Responsive**: Uses Positioned.directional to adapt to different language directions

## Spacing and Layout Design

The entire UI uses consistent spacing design:

- **Avatar bottom spacing**: `const SizedBox(height: 10)`
- **Username bottom spacing**: `const SizedBox(height: 10)`
- **Icon row bottom spacing**: `const SizedBox(height: 20)`
- **Default button spacing**: Through Column's default spacing

## Responsive Design Considerations

1. **Fixed width layout**: Uses 400 pixel maximum width for consistent experience across screen sizes
2. **Scroll support**: SingleChildScrollView supports scrolling when content exceeds screen
3. **Touch targets**: All buttons have adequate touch areas
4. **Text direction support**: Uses Positioned.directional and Directionality.of for RTL language adaptation
5. **Keyboard handling**: GestureDetector dismisses keyboard when tapping blank areas

This UI design fully considers user experience with intuitive layout, appropriate spacing, smooth animations, and good accessibility.
