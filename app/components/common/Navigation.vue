<script> // eslint-disable-line vue/multi-word-component-names
import {
  cocoBaseURL,
  CODECOMBAT,
  getQueryVariable,
  isChinaOldBrowser,
  isCodeCombat,
  isOzaria,
  OZARIA,
  ozBaseURL
} from 'core/utils'
import AnnouncementModal from '../../views/announcement/announcementModal'
import AnnouncementNav from '../../views/announcement/AnnouncementNav'
import { mapActions, mapGetters } from 'vuex'

/**
 * Unified navigation bar component between CodeCombat and Ozaria.
 */
export default Vue.extend({
  computed: {
    ...mapGetters('announcements', [
      'announcements',
      'unread',
      'announcementInterval',
      'announcementModalOpen',
      'announcementDisplay'
    ]),
    isChinaOldBrowser () {
      return isChinaOldBrowser()
    },

    isCodeCombat () {
      return isCodeCombat
    },

    isOzaria () {
      return isOzaria
    },

    cocoBaseURL () {
      return cocoBaseURL()
    },

    ozBaseURL () {
      return ozBaseURL()
    },

    hideNav () {
      return getQueryVariable('landing', false)
    }
  },

  created () {
    // Bind the global values to the vue component.
    this.me = me
    this.document = window.document
    this.serverConfig = window.serverConfig
    this.serverSession = window.serverSession
    this.CODECOMBAT = CODECOMBAT
    this.OZARIA = OZARIA
  },
  mounted () {
    setTimeout(() => {
      this.checkAnnouncements('fromNav')
      if (!this.announcementInterval) { // todo: using websocket to get new announcements
        this.startInterval('fromNav')
      }
    }, 2000)
  },
  beforeUnmounted () {
    if (this.announcementInterval) { clearInterval(this.announcementInterval) }
  },
  methods: {
    ...mapActions('announcements', [
      'closeAnnouncementModal',
      'checkAnnouncements',
      'startInterval'
    ]),
    navEvent (e) {
      // Only track if user has clicked a link on the nav bar
      if (!e || !e.target || e.target.tagName !== 'A') {
        return
      }

      if (!window.tracker) {
        return
      }

      const clickedAnchorTag = e.target
      const action = `Link: ${clickedAnchorTag.getAttribute('href') || clickedAnchorTag.getAttribute('data-event-action')}`
      const properties = {
        category: 'Nav',
        // Inspired from the HomeView homePageEvent method
        user: me.get('role') || (me.isAnonymous() && 'anonymous') || 'homeuser'
      }

      window.tracker.trackEvent(action, properties)
    },

    /**
     * This is used to highlight nav routes we are currently on.
     * It can optionally also check if the user is on codecombat or ozaria.
     */
    checkLocation (route, host = undefined) {
      let hostCheck = true
      if (host === CODECOMBAT) {
        hostCheck = this.isCodeCombat
      } else if (host === OZARIA) {
        hostCheck = this.isOzaria
      }
      return hostCheck && document.location.href.search(route) >= 0
    },

    /**
     * Returns a codecombat url for a relative path.
     * If the user is already on codecombat, will return a relative URL.
     * If the user is on ozaria, will return an absolute url to codecombat.com
     *
     * Handles subdomains such as staging.ozaria.com, will return absolute path
     * to staging.codecombat.com
     *
     * The domains used in China are also handled, i.e. koudashijie
     */
    cocoPath (relativePath) {
      return `${this.cocoBaseURL}${relativePath}`
    },

    ozPath (relativePath) {
      return `${this.ozBaseURL}${relativePath}`
    },

    readAnnouncement () {
      return application.router.navigate('/announcements', { trigger: true })
    }
  },
  components: {
    AnnouncementModal,
    AnnouncementNav
  }
})
</script>

<template lang="pug">
  nav#main-nav.navbar.navbar-default.navbar-fixed-top.text-center(:class="/^\\/(league|play\\/ladder)/.test(document.location.pathname) ? 'dark-mode' : ''" @click="navEvent")
    announcement-modal(v-if="announcementModalOpen" @close="closeAnnouncementModal" :announcement="announcementDisplay")
    .container-fluid
      .row
        .col-md-12
          .navbar-header
            button.navbar-toggle.collapsed(data-toggle='collapse', data-target='#navbar-collapse' aria-expanded='false')
              span.sr-only {{ $t('nav.toggle_nav') }}
              span.icon-bar
              span.icon-bar
              span.icon-bar
            a.navbar-brand(v-if="me.useTarenaLogo()" href="http://kidtts.tmooc.cn/ttsPage/login.html")
              picture
                source#logo-img.powered-by(srcset="/images/pages/base/logo.webp" type="image/webp")
                img#logo-img.powered-by(src="/images/pages/base/logo.png" alt="CodeCombat logo")
              img#tarena-logo(src="/images/pages/base/logo-tarena.png" alt="Tarena logo")
            a.navbar-brand(v-else-if="serverConfig.codeNinjas" href="/home")
              picture
                source#logo-img.powered-by(srcset="/images/pages/base/logo.webp" type="image/webp")
                img#logo-img.powered-by(src="/images/pages/base/logo.png" alt="CodeCombat logo")
              img.code-ninjas-logo(src="/images/pages/base/code-ninjas-logo-right.png" alt="Code Ninjas logo")
            a.navbar-brand(v-else-if="me.isTecmilenio()" href="/home")
              picture
                source#logo-img.powered-by(srcset="/images/pages/base/logo.webp" type="image/webp")
                img#logo-img.powered-by(src="/images/pages/base/logo.png" alt="CodeCombat logo")
              img.tecmilenio-logo(src="/images/pages/payment/tecmilenio-logo-2.png" alt="Tecmilenio logo")
            a.navbar-brand(v-else-if="me.showChinaResourceInfo()" href="/home")
              img#logo-img(src="/images/pages/base/logo-en+cn.png" alt="CodeCombat logo")
            a.navbar-brand(v-else :href="hideNav ? '#' : '/home'")
              picture
                source#logo-img(srcset="/images/pages/base/logo.webp" type="image/webp")
                img#logo-img(src="/images/pages/base/logo.png" alt="CodeCombat logo")

          .navbar-browser-recommendation.navbar-header(v-if="isChinaOldBrowser")
            .nav-spacer
              .navbar-nav
                a.text-p(href="https://www.google.cn/intl/zh-CN/chrome/") {{ $t('nav.browser_recommendation') }}

          #navbar-collapse.collapse.navbar-collapse
            .nav-spacer
            ul.nav.navbar-nav(v-if="!me.hideTopRightNav() && !hideNav")
              template(v-if="me.showChinaResourceInfo()")
                li
                  a.text-p(href="https://blog.koudashijie.com") {{ $t('nav.blog') }}

                li
                  a.text-p(data-event-action="Header Request Quote CTA", href="/contact-cn") {{ $t('new_home.request_quote') }}

              li(v-if="me.isAnonymous()")
                ul.nav.navbar-nav
                  li.dropdown.dropdown-hover
                    a.text-p(:href="isCodeCombat ? '/impact' : '/'", data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false" :class="isOzaria && 'text-teal'")
                      span {{ $t('nav.educators') }}
                      span.caret
                    ul(class="dropdown-menu")
                      li
                        a.text-p(:href="ozPath('/')")
                          span(:class="isOzaria && !checkLocation('/professional-development') && 'text-teal'") {{ $t('nav.ozaria_classroom') }}
                      li
                        a.text-p(:href="cocoPath('/impact')" :class="checkLocation('/impact', CODECOMBAT) && 'text-teal'") {{ $t('nav.codecombat_classroom') }}
                      li
                        a.text-p(:href="ozPath('/professional-development')")
                          span(:class="checkLocation('/professional-development') && 'text-teal'") {{ $t('nav.professional_development') }}

              li(v-if="!me.isStudent() && !me.isTeacher() && (me.get('country') !== 'hong-kong') && !me.isParentHome()")
                a.text-p(:class="checkLocation('/parents') && !checkLocation('/parents/signup') && 'text-teal'" :href="cocoPath('/parents')") {{ $t('nav.parent') }}

              li(v-if="me.isParentHome()")
                a.text-p(:class="checkLocation('/parents/dashboard') && 'text-teal'" :href="me.hasNoVerifiedChild() ? cocoPath('/parents/add-another-child') : cocoPath('/parents/dashboard')") {{ $t('nav.dashboard') }}

              li
                a.text-p(:class="checkLocation('/league') && 'text-teal'" :href="cocoPath('/league')") {{ $t('nav.esports') }}

              li(v-if="me.isTeacher()")
                ul.nav.navbar-nav
                  li.dropdown.dropdown-hover
                    a.dropdown-toggle.text-p(href="/teachers/classes", data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false")
                      span {{ $t('nav.dashboard') }}
                      span.caret
                    ul(class="dropdown-menu")
                      li
                        a.text-p(:href="ozPath('/teachers/classes')")
                          span(:class="checkLocation('/teachers/classes', OZARIA) && 'text-teal'") {{ $t(`nav.ozaria${me.isSchoolAdmin()?'_teacher':''}_dashboard`) }}
                      li
                        a.text-p(:class="checkLocation('/teachers/classes', CODECOMBAT) && 'text-teal'" :href="cocoPath('/teachers/classes')") {{ $t(`nav.codecombat${me.isSchoolAdmin()?'_teacher':''}_dashboard`) }}

                      li(v-if="me.isSchoolAdmin()")
                        a.text-p(:href="ozPath('/school-administrator')")
                          span(:class="checkLocation('/school-administrator', OZARIA) && 'text-teal'") {{ $t(`nav.ozaria_admin_dashboard`) }}
                      li(v-if="me.isSchoolAdmin()")
                        a.text-p(:class="checkLocation('/school-administrator', CODECOMBAT) && 'text-teal'" :href="cocoPath('/school-administrator')") {{ $t(`nav.codecombat_admin_dashboard`) }}

              li(v-else-if="me.isStudent()")
                ul.nav.navbar-nav
                  li.dropdown.dropdown-hover
                    a.dropdown-toggle.text-p(href="#", data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false")
                      span {{ $t('nav.my_courses') }}
                      span.caret
                    ul(class="dropdown-menu")
                      li
                        a.text-p(:href="ozPath('/students')")
                          span(:class="checkLocation('/students', OZARIA) && 'text-teal'") {{ $t('nav.ozaria_classroom') }}
                      li
                        a.text-p(:class="checkLocation('/students', CODECOMBAT) && 'text-teal'" :href="cocoPath('/students')") {{ $t('nav.codecombat_classroom') }}

              li(v-if="!me.isAnonymous() && !me.isStudent() && !me.isTeacher()")
                a.text-p(:href="cocoPath('/play')") {{ $t('common.play') }}

            ul.nav.navbar-nav
              li.dropdown
                a.dropdown-toggle.text-p(href="#", data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false")
                  //- string replaced in RootView
                  span.language-dropdown-current Language
                  span.caret
                ul(class="dropdown-menu language-dropdown")

            ul.nav.navbar-nav(v-if="!me.isAnonymous()")
              li(v-if="me.isTarena()")
                a.text-p#logout-button {{ $t('login.log_out') }}
              li.dropdown(v-else)
                a.dropdown-toggle.text-p(href="#", data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false")
                  img.img-circle.img-circle-small.m-r-1(:src="me.getPhotoURL()" :class="{'border-navy': me.isTeacher()}")
                  span.unreadMessage(v-if="unread")
                  span {{ $t('nav.my_account') }}
                  span.caret
                ul.dropdown-menu.pull-right
                  li.user-dropdown-header.text-center.hidden-xs.hidden-sm
                    a(:href="cocoPath(`/user/${me.getSlugOrID()}`)")
                      img.img-circle(:src="me.getPhotoURL()" :class="me.isTeacher() ? 'border-navy' : ''")
                      h5 {{ me.broadName() }}
                  //- Account links
                  li(v-if="isCodeCombat")
                    a.account-dropdown-item(:href="cocoPath(`/user/${me.getSlugOrID()}`)") {{ $t('nav.profile') }}
                  li
                    a.account-dropdown-item(href="/account/settings") {{ $t('play.settings') }}
                  li.dropdown.dropleft.dropdown-hover(v-if="true || unread")
                    a.account-dropdown-item.dropdown-toggle(href="#", data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false" @click="readAnnouncement")
                      span.caret(v-if="this.announcements.length")
                      span {{ $t('announcement.notifications') }}
                      span.unread(v-if="unread") {{ unread }}
                    announcement-nav.announcement-nav(v-if="this.announcements.length")
                  li(v-if="isCodeCombat && (me.isAdmin() || !(me.isTeacher() || me.isStudent() || me.freeOnly()))")
                    a.account-dropdown-item(href="/account/payments") {{ $t('account.payments') }}
                  li(v-if="isCodeCombat && (me.isAdmin() || !(me.isTeacher() || me.isStudent() || me.freeOnly()) || me.hasSubscription())")
                    a.account-dropdown-item(href="/account/subscription") {{ $t('account.subscription') }}
                  li(v-if="isCodeCombat && (me.isAdmin() || (me.get('emailVerified') && (me.isTeacher() || me.isParentHome() || (!me.get('role') && !me.isAnonymous()))))")
                    a.account-dropdown-item#manage-billing(href="/payments/manage-billing", target="_blank") {{ $t('account.manage_billing') }}
                  li(v-if="me.isAPIClient()")
                    a.account-dropdown-item(href="/partner-dashboard", target="_blank") {{ $t('nav.api_dashboard') }}
                  li(v-if="me.isAdmin() || me.isOnlineTeacher() || me.isParentAdmin()")
                    a.account-dropdown-item(href="/admin") {{ $t('account_settings.admin') }}
                  li(v-if="me.isAdmin() || me.isOnlineTeacher()")
                    a.account-dropdown-item(href="/event-calendar/classes") {{ $t('events.calendar') }}
                  li(v-if="serverSession && serverSession.amActually")
                    a.account-dropdown-item#nav-stop-spying-button(href="#") {{ $t('login.stop_spying') }}
                  li(v-if="me.isTeacher()")
                    a.account-dropdown-item#nav-student-mode(href="#") {{ $t('login.test_as_student') }}
                  li(v-else-if="serverSession && serverSession.switchingUserActualId && me.isTestStudent()")
                    a.account-dropdown-item#nav-stop-switching-button(href="#") {{ $t('login.stop_switching') }}
                  li
                    a.account-dropdown-item#logout-button(href="#") {{ $t('login.log_out') }}

            ul.nav.navbar-nav.text-p.login-buttons(v-if="me.isAnonymous() && !hideNav")
              li
                button#create-account-link.signup-button(data-event-action="Header Sign Up CTA") {{ $t('signup.sign_up') }}
              li
                button#login-link.login-button(data-event-action="Header Login CTA") {{ $t('signup.login') }}
</template>

<style lang="scss" scoped>
/* These styles are global. This is required so bootstrap.... :( */
@import "app/styles/bootstrap/variables";
@import "app/styles/mixins";
@import "app/styles/style-flat-variables";

#main-nav.navbar {
  background: white;
  margin-bottom: 0;
  white-space: nowrap; // prevent home icon from going under brand
  box-shadow: unset;
  font-weight: 400;

  @media print {
    display: none;
  }

  h5 {
    font-family: "Arvo", serif;
    font-weight: bold;
    font-size: 20px;
    line-height: 31px;
    font-variant: normal;
    color: black;
    margin: 0;
  }

  p, .text-p, .text-p button {
    font-family: $body-font;
    font-size: 18px;
    font-weight: 400;
    letter-spacing: 0.75px;
    line-height: 26px;
  }

  #create-account-link {
    background-color: $teal-dark;
    color: white;
    border: 1px solid $teal-dark;
    border-radius: 4px 0 0 4px;
    width: 131px;

    &:hover {
      background-color: #2DCEC8;
      border: 1px solid #2DCEC8;
      color: $teal-dark;
      transition: background-color .35s, border .35s;
    }
  }

  #login-link {
    width: 94px;
    border: 1px solid $teal;
    border-radius: 0 4px 4px 0;
    /*color: $teal;*/
    /* too faint for WCAG AAA */
    color: #16837f; /* increased contrast by lowering luminance */
    background: transparent;

    &:hover {
      background-color: #1FBAB4;
      color: white;
      transition: color .35s, background-color .35s;
    }
  }

  .nav-spacer {
    height: 12px;
  }

  .navbar-browser-recommendation {
    margin-left: 1em;
    padding-top: 15px;

    a {
      font-size: 16px;
      padding: 10px 15px;
      float: left;

      &:hover {
        color: $teal;
        text-decoration: none;
      }
    }
  }

  .login-buttons {
    margin: 2px 70px 0px 10px;
    @media (max-width: $screen-md-min) {
      display: inline-block;
      margin: 2px 10px 29.5px;
    }
    @media (max-width: $wider-breakpoint) {
      margin-right: 10px;
    }

    & li {
      display: inline-block;
    }

    & button {
      line-height: 20px;
    }
  }

  a.navbar-brand {
    padding: 14px 0 16px 70px;
    margin: 0px;
    @media (max-width: $wider-breakpoint) {
      padding-left: 10px;
    }

    #logo-img {
      height: 40px;

      &.powered-by {
        height: 30px;
        width: auto;
        margin-top: -5px;
      }
    }

    .code-ninjas-logo, #tarena-logo, .tecmilenio-logo {
      height: 40px;
      width: auto;
      margin-right: 10px;
    }
  }

  .navbar-toggle {
    color: black;
    margin: 30px 70px 0;
    border-color: $navy;
    @media (max-width: 767px) {
      margin: 15px 10px 0;
    }

    .icon-bar {
      background-color: $navy;
    }
  }

  @media (min-width: $grid-float-breakpoint) {
    #navbar-collapse {
      float: right;
    }

    .dropdown-menu {
      max-width: 330px;
      overflow-x: visible;
    }
  }

  .language-dropdown {
    max-height: 60vh;
    overflow-y: auto;
    left: -55px;
  }

  #navbar-collapse {
    max-height: 100vh;

    .text-teal {
      /*color: $teal;*/
      /* too faint for WCAG AAA */
      color: #16837f; /* increased contrast by lowering luminance */
      font-weight: 600; /* increased contrast by increasing weight */
    }
  }

  .nav > li > a, .nav > li > button {
    // TODO: Move this to bootstrap variables for navbars

    // TODO: getting overridden by .navbar .nav > li > a for some reason
    font-family: $body-font;
    text-shadow: unset;
    padding: 10px 15px;
    @media (max-width: $wider-breakpoint) {
      padding: 10px 10px;
    }

    color: $navy;

    &:hover {
      color: $teal;
    }
  }

  // TODO: what is this for?
  .nav > li.disabled > a, .nav > li.disabled > button {
    color: black;

    &:hover {
      background: white;
      color: black;
      cursor: default;
    }
  }

  .new-pill {
    font-size: 16px;
    font-weight: 600;
    background-color: #ff76c1;
    border-radius: 14px;
    padding: 4px;
    margin-left: 5px;
  }

  .dropdown-hover .dropdown-menu {
    padding: 0;
  }

  @media (min-width: $grid-float-breakpoint) {
    .dropdown-hover:hover {
      & > ul {
        /* Allows for mouse over to expand dropdown */
        display: unset;
      }
    }
  }

  .dropdown-hover .dropdown-menu li a {
    height: 50px;
    display: flex;
    align-items: center;
    color: #0E4C60;
  }

  @media (max-width: $grid-float-breakpoint) {
    .nav > li > a, .nav > li > button {
      padding: 10px 20px;
      height: 45px;
    }
    .language-dropdown-item {
      color: $navy;
    }
    .account-dropdown-item {
      color: $navy;
    }

    .dropdown-hover .dropdown-menu li a {
      justify-content: center;
    }

    .dropdown-menu.pull-right {
      /* Important required for bootstrap overwriting */
      float: unset !important;
    }
  }

  // TODO: still used?
  .img-circle {
    border: $gold 8px solid;
    width: 98px;
    height: 98px; // Includes the border
  }

  .img-circle-small {
    border: $gold 3px solid;
    width: 33px;
    height: 33px;
  }

  // For teacher avatars
  .border-burgundy {
    border-color: $navy;
  }

  .border-navy {
    border-color: $navy;
  }

  span.unreadMessage {
    width: 5px;
    height: 5px;
    position: absolute;
    top: 10px;
    left: 45px;
    border-radius: 50%;
    background-color: $yellow;
    box-shadow: 0 0 2px 2px $yellow;
  }

  .dropleft {
    .announcement-nav {
      position: absolute;
      left: auto;
      right: 100%;
      top: 0;
    }

    .caret {
      transform: rotate(90deg);
    }
  }

  span.unread {
    width: 1.2em;
    height: 1.2em;
    margin-left: 1em;
    line-height: 1.2em;
    border-radius: 50%;
    background-color: $yellow;
    color: white;
    display: inline-block;
    margin-left: 0.5em;
  }

}

nav#main-nav.navbar.dark-mode {
  background-color: #0C1016;

  .nav > li > a {
    color: #FCBB00;

    &:hover {
      color: #FF39A6;
    }
  }

  .dropdown-menu {
    background-color: white;
  }

  #create-account-link {
    background-color: #FCBB00;
    border: 1px solid #FCBB00;
    color: #0C1016;

    &:hover {
      background-color: #FF39A6;
      border: 1px solid #FF39A6;
    }
  }

  #login-link {
    color: #FCBB00;
    border: 1px solid #FCBB00;

    &:hover {
      background-color: #FF39A6;
      border: 1px solid #FF39A6;
      color: #0C1016;
    }
  }
}
</style>
