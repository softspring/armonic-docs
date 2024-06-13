# Create a new media

Symfonic CMS empowers you to integrate multimedia elements into your content, adding visual elements to your web pages. Whether it's images, videos, or icons, Symfonic CMS provides a simple process for creating and managing your media assets.

## Media Creation Journey

- Access the Media section: Navigate to the "Medias" section within the Symfonic CMS administration panel. This serves as your central repository for managing all your media files.

- Locate the **"New Media"** button, often found near the top right corner of the Media section. Clicking on this button will launch the media creation process.

  - **Selecting the Media Type**: Symfonic CMS presents a variety of media types to suit your creative needs. Choose from "Background Image," "Content Image (PNG)," "Content Image (JPG)," "Icon Image," "Content Video," or any other media types defined by your CMS. Carefully consider the intended use of the media when making your selection. 
  - **Filling in the Media Details**:

    - Provide a descriptive **"Name"** that accurately reflects the media's content, making it easy to identify and organize.
    - Craft a concise **"Description"** that serves as alternative text for the media. This ensures accessibility for visually impaired users and enhances search engine optimization.
    - Utilize the *"Main media"* field to select the media file you wish to add. Ensure the file format aligns with the chosen media type.
    - Additional File Uploads: For certain media types, such as images with different breakpoints, you may need to upload multiple files. Symfonic CMS will guide you through this process.

  - With a click on the **"Upload media"** button, your media creation journey comes to a successful end. **Your media asset is now ready to be incorporated into your website's content**.

## Show media details

The media list in Symfonic CMS offers a centralized section for managing and exploring your uploaded media assets. 

By clicking on a media element, you can access media information, all generated versions of the media and the Twig code that you can use in your templates if necessary.

`{{ media|sfs_media_render_image('sm', {'class':'img-sm'}) }}`
