# Disable the Firefox Tab Bar

As a user of the Tree Style Tab extension, I want to disable the default tab bar in Firefox.

- Figure out your profile folder (Help -> Troubleshooting Information -> Profile Folder)
- Create a folder `chrome` in the Firefox profile folder
- Add a `userChrome.css` file to the `chrome` folder.

    ```
    #TabsToolbar {
        visibility: collapse !important;
    }

    #sidebar-box[sidebarcommand="treestyletab_piro_sakura_ne_jp-sidebar-action"] #sidebar-header {
        display: none;
    }
    ```

    The first snippet removes the tab bar. The second snippets hides the title from the Tree Style Tab extension.

- In Firefox, open `about:config` and change the value of these two properties: 

  - `toolkit.legacyUserProfileCustomizations.stylesheets` -> `true`
  - `browser.tabs.drawInTitlebar` -> `false`

- Restart Firefox.
