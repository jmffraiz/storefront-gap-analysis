+------------------------------------------------------------------------------------------------------+
| Metadata                                                                                             |
+-----------------------+------------------------------------------------------------------------------+
| Template              | guides                                                                       |
+-----------------------+------------------------------------------------------------------------------+
| Category              | Build                                                                        |
+-----------------------+------------------------------------------------------------------------------+
| Image                 | ![][image0]                                                                  |
+-----------------------+------------------------------------------------------------------------------+
| Campaign: webcurrents | https\://main--helix-website--adobe.aem.page/developer/tutorial-from-youtube |
+-----------------------+------------------------------------------------------------------------------+

+-------------------+
| Section Metadata  |
+---------+---------+
| style   | content |
+---------+---------+

![][image0]

# Getting Started – Developer Tutorial

This tutorial will get you up-and-running with a new Adobe Experience Manager (AEM) project. In ten to twenty minutes, you will have created your own site and be able to create, preview, and publish your own content, styling, and add new blocks.

Prerequisites:

1. You have a GitHub account, and understand Git basics.
2. You understand the basics of HTML, CSS, and JavaScript.
3. You have Node/`npm` installed for local development.

This tutorial uses macOS, Chrome, and Visual Studio Code as the development environment and the screenshots and instructions reflect that setup. You can use a different operating system, browser, and code editor, but the UI you see and steps you must take may vary accordingly.

TIP: If you would like to **get your AEM project started the fastest way Adobe offers with content you already know,** **[use the AEM Modernization Agent.](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/ai-in-aem/agents/modernization/getting-started)** Just create your boilerplate repository and then collaborate with the agent to import your site.

## Get started with the boilerplate repository template

+-----------------------------------------------------------------------------------+
| Video                                                                             |
+-----------------------------------------------------------------------------------+
| <https://main--helix-website--adobe.aem.page/developer/videos/tutorial-step1.mp4> |
+-----------------------------------------------------------------------------------+

The fastest and easiest way to get started following AEM best practices is to create your repository using the Boilerplate GitHub repository as a template.\
\
<https://github.com/adobe/aem-boilerplate>

![][image1]

Click the `Use this template` button and select `Create a new repository`, and select the user org that owns the repository

![][image2]

We recommend that the repository is [set to public](https://www.aem.live/docs/dev-collab-and-good-practices#keep-the-repository-public).

The only remaining step in GitHub is to install the [AEM Code Sync GitHub App](https://github.com/apps/aem-code-sync) on your repository by visiting this link: <https://github.com/apps/aem-code-sync/installations/new>

\
![][image3]

In the `Repository access` settings of the AEM Code Sync App, make sure you select `Only select Repositories` (not `All Repositories`). Then select your newly created repository, and click `Save`.

**Note**: If you are using Github Enterprise with IP filtering, you can add the following IP to the allow list: `3.227.118.73`

![][image4]

Congratulations! You have a new website running on `https://<branch>--<repo>--<owner>.aem.page/` In the example above that’s `https://main--mysite--aemtutorial.aem.page/`

\
![][image5]

## Edit, Preview and Publish your content

Navigate to Author on <https://da.live/> and find the example content.

![][image6]

Edit content and `preview` and `publish` content as needed. For more information on authoring see <https://da.live/docs>.\
![][image7]

## Install Sidekick

\
To interact with AEM as an author across environments we strongly recommend installing the Sidekick Chrome extension. Find the [Chrome extension in the Chrome Web Store](https://chromewebstore.google.com/detail/aem-sidekick/igkmdomcgoebiipaifhmpfjhbjccggml).

![][image8]

After adding the extension to Chrome, don’t forget to pin it, this will make it easier to find it.

![][image9]

## Start developing styling and functionality

+-----------------------------------------------------------------------------------+
| Video                                                                             |
+-----------------------------------------------------------------------------------+
| <https://main--helix-website--adobe.aem.page/developer/videos/tutorial-step4.mp4> |
+-----------------------------------------------------------------------------------+

To get started with development, it is easiest to install the [AEM Command Line Interface](https://main--helix-website--adobe.aem.page/developer/cli-reference) (CLI) and clone your repo locally through using the following.

```
npm install -g @adobe/aem-cli
git clone https://github.com/<owner>/<repo>
```

\
\
From there change into your project folder and start your local development environment using the following.

```
cd <repo>
aem up
```

\
\
This opens `http://localhost:3000/` and you are ready to make changes.\
A good place to start is in the `blocks` folder which is where most of the styling and code lives for a project. Simply make a change in a `.css` or `.js `and you should see the changes in your browser immediately.

Once you are are ready to push your changes, simply use Git to add, commit, and push and your code to your preview (`https://<branch>--<repo>--<owner>.aem.page/`) and production (`https://<branch>--<repo>--<owner>.aem.live/`) sites.

**That’s it, you made it! Congrats, your first site is up and running. If you need help in the tutorial, please** **[join our Discord channel](https://discord.gg/aem-live)** **or** **[get in touch with us.](https://www.aem.live/business/reachout)**\
\
**To get you to a live website as fast as possible, this tutorial uses Document Authoring as the content source.** **[Universal Editor can also be configured for your project](https://docs.da.live/developers/reference/universal-editor#enable-your-project-for-universal-editor), to provide a WYSIWYG + Form-based authoring option.**

**NOTE:** **[Edge Delivery Services supports multiple content sources including Google Drive, Microsoft Sharepoint and AEM.](https://main--helix-website--adobe.aem.page/docs/authoring-guide)**

## Next Steps

Now that your site is up and running, choose how you want to continue:

- **Build your first block** - Create, style, and [deploy a custom block from scratch](https://www.aem.live/docs/exploring-blocks).
- **Build with AI** - Configure your project for [AI-assisted block creation and development.](https://www.aem.live/developer/ai-coding-agents#adding-skills-to-your-project)

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Pagination (Contained)                                                                                                                                                                   |
+----------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------+
| :icon-arrow: Previous                                                | Up Next :icon-arrow:                                                                                              |
|                                                                      |                                                                                                                   |
| ### [Build](https://main--helix-website--adobe.hlx.page/docs/#build) | ### [Anatomy of an AEM Project](https://main--helix-website--adobe.hlx.page/developer/anatomy-of-a-helix-project) |
+----------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------+

[image0]: https://main--helix-website--adobe.aem.page/media_1d00989ba18e942fbddc9bb108add01e153029f22.png#width=1600&height=1200

[image1]: https://main--helix-website--adobe.aem.page/media_1fdc7a94cd82e119b3c87d741d365731d7ccbcfcf.png#width=1600&height=1210

[image2]: https://main--helix-website--adobe.aem.page/media_19dfc49f63068dfa832f49743fa98f07379944c44.png#width=1600&height=1210

[image3]: https://main--helix-website--adobe.aem.page/media_1b900be829ffd16fa59db8cfc234bf84541565758.png#width=1600&height=1210

[image4]: https://main--helix-website--adobe.aem.page/media_196c384fa8247a89af448ee289b26711fd3c2314e.png#width=1600&height=1516

[image5]: https://main--helix-website--adobe.aem.page/media_131ece3853f2b2c1a95b9a8508d0a9e12cfeaeee3.png#width=1600&height=1210

[image6]: https://main--helix-website--adobe.aem.page/media_14d555da62ecc834d28567b16d941ab2b780be26b.png#width=1600&height=1248

[image7]: https://main--helix-website--adobe.aem.page/media_1f2996ba932ac1980183debd610b8f3ddca4e1e97.png#width=1600&height=1248

[image8]: https://main--helix-website--adobe.aem.page/media_1260919064b2b806e66abdd15fd2d2419a0ea6964.png#width=1600&height=1210

[image9]: https://main--helix-website--adobe.aem.page/media_15dd9adf32a7bf3b707b4dc42687e1d644b5a556e.png#width=1600&height=1210
