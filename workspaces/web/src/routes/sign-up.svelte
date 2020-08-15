<script>
  import db from "../db/db.js"
  import settings from "../settings"
  import NavBar from "../components/NavBar.svelte"
  import Button from "lluis/Button"
  import FormField from "lluis/FormField"
  import { _ } from "svelte-i18n"

  let loading = false

  let username = ""
  let email = ""
  let password = ""
  let password_confirmation = ""
  let license_accepted = false
  let errors = {}

  const emailRegexp = /^(([^<>()\[\]\.,;:\s@\"]+(\.[^<>()\[\]\.,;:\s@\"]+)*)|(\".+\"))@(([^<>()[\]\.,;:\s@\"]+\.)+[^<>()[\]\.,;:\s@\"]{2,})$/i

  const validateUsername = () => {
      if (!username) {
          errors = {
              ...errors,
              username: $_('signup.username_absent'),
          }

          return
      }

      if (username.length < 4) {
          errors = {
              ...errors,
              username: $_('signup.username_too_short'),
          }

          return
      }
  }

  const validateEmail = () => {
      if (!email) {
          errors = {
              ...errors,
              email: $_('signup.email_absent'),
          }

          return
      }

      if (!emailRegexp.test(email)) {
          errors = {
              ...errors,
              email: $_('signup.email_invalid'),
          }

          return
      }
  }

  const validatePassword = () => {
      if (!password) {
          errors = {
              ...errors,
              password: $_('signup.password_absent'),
          }

          return
      }

      if (password.length < 6) {
          errors = {
              ...errors,
              password: $_('signup.password_too_short'),
          }

          return
      }

      if (!password_confirmation) {
          errors = {
              ...errors,
              password_confirmation: $_('signup.password_confirmation_absent'),
          }

          return
      }

      if (password !== password_confirmation) {
          errors = {
              ...errors,
              password_confirmation: $_('signup.password_mismatch'),
          }

          return
      }
  }

  const validateLicense = () => {
      if (!license_accepted) {
          errors = {
              ...errors,
              license: $_('signup.license_not_accepted')
          }

          return
      }

      if (username.length < 4) {
          errors = {
              ...errors,
              username: $_('signup.username_too_short'),
          }

          return
      }
  }

  const handleTestingFakes = () => {
      if (window._test_fake_signup) {
          if (window._test_user_already_exists) {
              errors = {
                  ...errors,
                  _form: $_('signup.username_already_exists'),
              }
              return
          }
      }
  }

  let handleSignUp
  $: {
      handleSignUp = async () => {
          loading = true
          errors = {}
                   validateUsername()
                   validateEmail()
                   validatePassword()
                   validateLicense()
                   handleTestingFakes()
                   const isFormValid = Object.keys(errors).length === 0

                   if (process.browser === true) {
                       if (window._test_fake_signup) {
                           setTimeout(function () {
                               if (isFormValid === true) {
                                   loading = false
                                   window.location = "/sign-up-success"
                               } else {
                                   loading = false
                               }
                           }, 500)
                       } else {
                           if (isFormValid) {
                               fetch(settings.database.signUpEndpoint, {
                                   method: "post",
                                   headers: {
                                       "Content-Type": "application/json",
                                   },
                                   body: JSON.stringify({
                                       username,
                                       email,
                                       password,
                                   }),
                               })
                               .then((data) => data.json())
                               .then(({ success, error }) => {
                                   if (success) {
                                       loading = false
                                       window.location = "/sign-up-success"
                                   } else {
                                       loading = false
                                       if (error.code === "invalid-payload") {
                                           errors = error.details
                                       } else {
                                           errors = { _form: "Server error" }
                                       }
                                   }
                               })
                           } else {
                               loading = false
                           }
                       }
                   }
                   }
      }
</script>

<svelte:head>
  <title>{$_('signup.title')} - LibreLingo</title>
</svelte:head>

<NavBar dark />

<section class="section">

  <div class="container">

    <form on:submit|preventDefault="{handleSignUp}">
      <h2 class="is-size-2">{$_('signup.title')}</h2>

      <FormField
        name={$_('signup.username')}
        icon="user"
        id="username"
        formStatus="{errors}"
        bind:value="{username}" />
      <FormField
        name={$_('signup.email')}
        icon="envelope"
        id="email"
        formStatus="{errors}"
        bind:value="{email}" />
      <FormField
          name={$_('signup.password')}
        icon="lock"
        id="password"
        type="password"
        formStatus="{errors}"
        bind:value="{password}" />
      <FormField
        name={$_('signup.password_repeat')}
        icon="lock"
        id="password_confirmation"
        type="password"
        formStatus="{errors}"
        bind:value="{password_confirmation}" />

      <div class="field">
        <div class="control">
          <label class="checkbox">
            <input
              type="checkbox"
              name="license"
              id="license"
                bind:checked="{license_accepted}" />
            {@html $_('signup.license_agreement',
                {values: {
                    tos: `<a href="/tos">${$_('signup.tos')}</a>`,
                    gnu_license: `<a href="/license">${$_('signup.gnu_license')}</a>`
                }
              }) }
          </label>
        </div>
        {#if errors['license'] != null}
          <p class="help is-danger">{errors['license']}</p>
        {/if}
      </div>

      {#if errors._form != null}
        <p class="help is-danger">{errors._form}</p>
      {/if}

      <Button
        on:click="{handleSignUp}"
        {loading}
        asHref="/sign-up-success"
        submit>
        {$_('signup.title')}
      </Button>
    </form>
  </div>

</section>
