# Profile.dart User Profile Management Features

## Avatar Management Features

### getPhotoURLFromUser Method

```dart 344:384:packages/firebase_auth/firebase_auth/example/lib/profile.dart
  Future<String?> getPhotoURLFromUser() async {
    String? photoURL;

    // Update the UI - wait for the user to enter the SMS code
    await showDialog<String>(
      context: context,
      barrierDismissible: false,
      builder: (context) {
        return AlertDialog(
          title: const Text('New image Url:'),
          actions: [
            ElevatedButton(
              onPressed: () {
                Navigator.of(context).pop();
              },
              child: const Text('Update'),
            ),
            OutlinedButton(
              onPressed: () {
                photoURL = null;
                Navigator.of(context).pop();
              },
              child: const Text('Cancel'),
            ),
          ],
          content: Container(
            padding: const EdgeInsets.all(20),
            child: TextField(
              onChanged: (value) {
                photoURL = value;
              },
              textAlign: TextAlign.center,
              autofocus: true,
            ),
          ),
        );
      },
    );

    return photoURL;
  }
```

This method prompts the user to enter a new avatar URL through a dialog:

1. Shows a non-dismissible dialog (`barrierDismissible: false`)
2. Contains title "New image Url:"
3. Provides two buttons: Update and Cancel
4. Uses TextField for user to input new image URL
5. Returns the entered URL or null (when canceled)

### Avatar Update UI Component

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

Avatar display component includes:

1. **CircleAvatar**: Circular avatar display
   - Uses user avatar URL or default placeholder
   - 60 pixel radius

2. **Edit button**: Circular edit button positioned in bottom-right corner
   - Uses Material design with rounded corners
   - Triggers avatar update logic when tapped
   - Calls `user.updatePhotoURL(photoURL)` to update Firebase user profile upon success

## Display Name Editing Features

### Display Name Input Field

```dart 146:159:packages/firebase_auth/firebase_auth/example/lib/profile.dart
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

Display name editing component:

- Center-aligned text input field
- Borderless design consistent with page styling
- Hint text: "Click to add a display name"
- Bound to state management through controller

### Save Button Logic

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

Save button characteristics:

- Positioned in top-right corner of page
- Uses AnimatedSwitcher for smooth show/hide animations
- Only displays when display name has changed
- Disabled during loading
- Calls `updateDisplayName` method when tapped

## Email Verification Features

```dart 179:184:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      TextButton(
                        onPressed: () {
                          user.sendEmailVerification();
                        },
                        child: const Text('Verify Email'),
                      ),
```

Email verification button:

- Calls Firebase Auth's `sendEmailVerification()` method
- Sends verification email to user's mailbox
- Simple trigger operation without complex state management

## Apple Sign-in Token Revocation Features

```dart 192:210:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      TextButton(
                        onPressed: () async {
                          if (AuthGate.appleAuthorizationCode != null) {
                            // The `authorizationCode` is on the user credential.
                            // e.g. final authorizationCode = userCredential.additionalUserInfo?.authorizationCode;
                            await FirebaseAuth.instance
                                .revokeTokenWithAuthorizationCode(
                              AuthGate.appleAuthorizationCode!,
                            );
                            // You may wish to delete the user at this point
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

Apple token revocation features:

1. **Condition check**: Ensures `appleAuthorizationCode` is not null
2. **Token revocation**: Calls `revokeTokenWithAuthorizationCode()` to revoke Apple sign-in token
3. **Cleanup**: Sets `authorizationCode` to null
4. **Error handling**: Prints error message if `authorizationCode` is null
5. **Comment note**: Mentions option to delete user account

## Sign Out Features

### _signOut Method

```dart 387:390:packages/firebase_auth/firebase_auth/example/lib/profile.dart
  /// Example code for sign out.
  Future<void> _signOut() async {
    await auth.signOut();
    await GoogleSignIn().signOut();
  }
```

Sign out implementation:

1. Calls Firebase Auth's `signOut()` method
2. Calls Google Sign In's `signOut()` method
3. Ensures simultaneous cleanup of both Firebase and Google sign-in states

### Sign Out Button

```dart 315:318:packages/firebase_auth/firebase_auth/example/lib/profile.dart
                      TextButton(
                        onPressed: _signOut,
                        child: const Text('Sign out'),
                      ),
```

Simple sign out button that calls `_signOut` method when tapped.

## Sign-in Provider Display

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

Displays corresponding icons based on user's sign-in providers:

- **Phone sign-in**: Shows phone icon
- **Email/password sign-in**: Shows mail icon
- **Google sign-in**: Shows Google icon (network image)

This document provides detailed information about all core user profile management features, including avatar updates, display name editing, email verification, Apple token management, sign out, and sign-in method display.
