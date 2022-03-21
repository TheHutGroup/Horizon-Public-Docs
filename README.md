# Horizon Public Docs

Most information can be found in the index file and other files. This is used to describe how to run this locally to verify and test changes before publishing.


To install, follow the commands here: https://jekyllrb.com/docs/installation/macos/

This will detail how to get the latest version of ruby and put it on your path. Once that's done the follow commands are needed.
```
gem install --user-install bundler jekyll
bundle install
bundle exec jekyll serve
```

This will spin the docs up locally and will be accessible here: http://127.0.0.1:4000/Horizon-Public-Docs/

For the current live version of the docs, these can be accessed here: https://thehutgroup.github.io/Horizon-Public-Docs/

## Contributing
Before contributing to this repo, it is worth setting your git config to use your do-not-reply github address:
https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-github-user-account/managing-email-preferences/setting-your-commit-email-address

NOTE: Do not use the global flag for setting user.email as that will update the email used in other repositories that you probably dont want
