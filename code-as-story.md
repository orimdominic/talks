---
theme: default
layout: intro
class: text-center
highlighter: shikiji
lineNumbers: true
# transition: slide-left
title: Writing Code like a Storyteller
author: Orim Dominic Adah
keywords: "clean code"
mdc: true
fonts:
  sans: 'Raleway'
  mono: 'Fira Code'
htmlAttrs:
  dir: 'ltr'
  lang: 'en'
---


# Writing Code like a Storyteller
<style>
  .subtitle {
    opacity: 0.8;
  }
</style>
<p class="text-7 pb-8">
  How to write code that does not make people cry
</p>


Orim Dominic<br>
Web Developer, Zedi Africa

---
layout: full
---
# Once upon a time...

---
layout: full
---
# Once upon a time...

<img src="/readable-and-clean-I.png" class="mt-8 ml-24 h-60 rounded shadow" />
<img src="/readable-and-clean-II.png" class="ml-24 h-27.5 rounded shadow" />

---
layout: quote
---
# Programs are meant to be read by humans... and for computers to execute.

<p class="text-6 text-right">
  Donald Knuth
</p>

---
layout: full
---
# What do computers read eventually?

<style>
  pre {
    font-size: 16px !important;
  }
</style>

```bash
1010101 1010 00100 101 111 111 1100 1110 001 1101 101

1010101 1010 1101 101

101 1010 11111 101 111 111 11 1110 101 1101 101

1011101 1010 00101 101 111 11 10001 111 101 1 101 111 10001 1110

1011 1010 101 101 111 111 10001 1 101 1101 101

1010101 1010 11111 101 111 111 10001 1001 101 1101 101

11 1010 11111 101 111 111 10001 1110 101 1101 101

1010101 1010  101 1101 101
```

---
layout: cover
---
<h1 class="font-extrabold">Primary Concerns</h1>

<section class="text-8 font-bold">
  <ul>
    <li class="py-4">
      <span>Functionality</span>
    </li>
    <li class="">
      <span>Maintainability</span>
    </li>
    <li class="op-50 pt-4">
      <span>Performance</span>
    </li>
  </ul>
</section>

---
layout: cover
---
<h1 class="font-extrabold">Primary Concerns</h1>

<section class="text-8 font-bold">
  <ul>
    <li class="op-50 py-4">
      <span>Functionality</span>
    </li>
    <li class="">
      <span>Maintainability</span>
    </li>
    <li class="op-50 pt-4">
      <span>Performance</span>
    </li>
  </ul>
</section>

---
layout: fact
---
# Show me the code!

---
layout: full
---

<style>
  pre {
    max-height: 500px;
  }
</style>

```js
/**
 * @param {Object} loginDto
 * @param {string} loginDto.email
 * @param {string} [loginDto.photoUrl]
 * @param {'apple' | 'google'} [loginDto.appleId]
 * @returns {Object} user
 */
async function updateUserOnLogin(dto) {
  let user;

  if (dto.loginProvider == "apple") {
    user = await findOneByappleId(dto.appleId);

    if (user?.isDisabled) {
      throw new HttpException("Unable to login");
    }

    if (user && !user.verified) {
      user = await updateUserAccountVerifiedStatus(user, true);
    }

    // Temporarily placing this here, because users don't have their apple user
    // id stored in the db now. When we are certain all apple users have their
    // appleId stored, we can remove this line
    if (!user) {
      user = await findOneByEmail(dto.email);

      if (user && dto.appleId) {
        user = await updateUserAppleId(user, dto.appleId);
      }

      if (user && !user.verified) {
        user = await updateUserAccountVerifiedStatus(user, true);
      }
    }
  } else {
    user = await findOneByEmail(dto.email);

    if (user?.isDisabled) {
      throw new HttpException("Unable to login");
    }

    // Temporarily placing this here, ensures pictures of existing users that
    // signed via google are pulled from google and saved (if they haven't
    // set a picture already)
    if (user && !user.photoUrl && dto.photoUrl) {
      user.photoUrl = dto.photoUrl;
      user = await updateUserDetails(user._id, user);
    }

    if (user && !user.verified) {
      user = await updateUserAccountVerifiedStatus(user, true);
    }
  }

  return user
}
```
---
layout: cover
---

## What's the story?

- initialise a `user` variable to value of `undefined` to hold the user account
- if `loginProvider` is `apple`, find the user account using the `appleId`
  - if the user account is found but is disabled, bounce the user
  - if the user account is found but is not verified, set it to verified (cos of the `appleId`)
  - if no `user` account from `appleId` find one from user `email`
  - if user account is found and `appleId` is not a falsy value set the user's appleId
  - if user account exists and user account is not verified, set it to verified
- if `loginProvider` is `google`
  - find the user by their email
  - if the user account is found but is disabled, bounce the user
  - if the user account is found and does not have their photo update the user's photoUrl to the one from `photoUrl`
  - if the user account is not verified, set it to verified
- return `user`

---
layout: full
---

<style>
  pre {
    max-height: 500px;
  }
</style>

```js {all|1,2|3-7|9-14|15-17|19-33}
async function updateUserOnLogin(dto) {
  let user = await findUserByEmail(dto.email);
  // * Consider failure cases first and get them out of the story and focus on
  //   other things for proper narration. No back and forth.
  if (!user) {
    user = await createUser(dto);
  }
  // * Let the code read like the grammar you speak daily
  if (user.isDisabled) { // A newly-created user will not be disabled, right?
    // * Handle failure cases early. Good for testing too.
    // * Use early returns. They help end functions early so you can focus on
    //   other things
    throw new HttpException("Unable to login");
  }
  // * Avoid nesting conditionals inside conditionals.
  //   It complicates the story and makes it hard to follow. Use && instead.
  if (!user.isVerified && dto.appleId) {
    await setUserAsVerified(user.email);
    // * Don't use your code comments to say WHAT is obviously happening
    //   Use it to say WHY it is happening
    //   Use identifiers/variable names to tell us WHAT is happening
    //                👇        👇          👇
    // Temporarily placing this here, because users don't have their
    // apple user id stored in the db now. When we are certain all apple users
    // have their appleId stored, we can remove this line
    await setUserAppleId(user.email, dto.appleId);
  }
  // Temporarily placing this here, ensures pictures of existing users
  // that signed via Google are pulled from Google and saved if they haven't
  // set a picture already
  if (!user.photoUrl && dto.photoUrl) {
    await updateUserDetails(user.id, { photoUrl: dto.photoUrl });
  }

  return user;
}
```

---
layout: cover
---

<style>
  pre {
    max-height: 500px;
  }
</style>

```js
async function updateUserOnLogin(dto) {
  let user = await findUserByEmail(dto.email);

  if (!user) {
    user = await createUser(dto);
  }

  if (user.isDisabled) {
    throw new HttpException("Unable to login");
  }

  if (!user.isVerified && dto.appleId) {
    await setUserAsVerified(user.email);
    await setUserAppleId(user.email, dto.appleId);
  }

  if (!user.photoUrl && dto.photoUrl) {
    await updateUserDetails(user.id, { photoUrl: dto.photoUrl });
  }

  return user;
}
```

---
layout: cover
---

```js
/**
 * @param {string} email
 * @param {string} [appleId]
 */
async function setUserAppleId(email, appleId) {
  // * Once again, handle failure cases early. Use early returns wherever you can
  if (!appleId) return;

  const updatedUser = await updateUserByEmail(email, { appleId });

  return updatedUser
}
```

---
layout: cover
---

<h1 class="font-extrabold">Key Takeaways</h1>

<section class="text-8 font-bold">
  <ul>
    <li class="py-8">
      <span>Name identifiers well</span>
    </li>
    <li class="">
      <span>Avoid deep nesting</span>
    </li>
    <li class="pt-8">
      <span>Handle unhappy paths early</span>
    </li>
  </ul>
</section>

---
layout: cover
---
<h1 class="font-extrabold text-20">Like reading a good story, reading code can be an enjoyable activity.</h1>


---
layout: default
---
<h1 class="font-extrabold">References and Further Study</h1>

- [Refactoring: Choosing abstractions](https://mykeels.medium.com/refactoring-choosing-abstractions-c8e9a681ae0d) by Micheal Ikechi

- All of Venkat Subramaniam's talks on Code Quality on Youtube


