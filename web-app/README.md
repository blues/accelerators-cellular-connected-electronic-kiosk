# Cellular-Connected Electronic Kiosk Web App

The Cellular-Connected Electronic Kiosk web app is intended to be an example for how a web app could be built with a greater focus on the acts of compressing the app and zipping it up, uploading it to a cloud, and then using the Notecard to download the ZIP file via cellular, unpack it, and run the app locally on the Raspberry Pi.

- [Cellular-Connected Electronic Kiosk Web App](#cellular-connected-electronic-kiosk-web-app)
  - [Brief Demo Web App Overview](#brief-demo-web-app-overview)
  - [How to Package Up the App in a ZIP File](#how-to-package-up-the-app-in-a-zip-file)
    - [Create a Fresh ZIP of the Demo App](#create-a-fresh-zip-of-the-demo-app)
  - [Upload the ZIP to a Cloud](#upload-the-zip-to-a-cloud)
    - [Create New S3 Bucket](#create-new-s3-bucket)
    - [Upload the ZIP to the S3 Bucket](#upload-the-zip-to-the-s3-bucket)
  - [Create a Notehub Proxy Route Pointing Towards the AWS Bucket](#create-a-notehub-proxy-route-pointing-towards-the-aws-bucket)
  - [Make Sure the Notehub Fleet Environment Variables are Set](#make-sure-the-notehub-fleet-environment-variables-are-set)
  - [Initiate the Download of the ZIP to the Raspberry Pi](#initiate-the-download-of-the-zip-to-the-raspberry-pi)
  - [Update the Notehub Env Vars and See the Changes Take Effect in the App](#update-the-notehub-env-vars-and-see-the-changes-take-effect-in-the-app)

## Brief Demo Web App Overview

This demo project is a simple web application built with HTML, CSS and vanilla JavaScript - it was built this way in an effort to keep the total app size small, and speed up download times to the Raspberry Pi.

The app expects a list of city names (either from its local [`data.js`](./data.js) file, or from the Notehub environment variable `kiosk_data`) and uses the [Weather API](https://www.weatherapi.com/) to fetch the weather forecast for each city, then display that data onscreen and scroll through each city's weather, changing the city displayed on screen every 4 seconds.   

https://user-images.githubusercontent.com/20400845/235251916-3e76d47a-0771-4d8e-94dd-77d09238d0cf.mov

## How to Package Up the App in a ZIP File

In order to make this app easy to download from an online location, it must first be stored in a compressed ZIP file.

For the demo app, there are two shell script files that help automate this process:

* [`generate.sh`](./generate.sh)
* [`package.sh`](../package.sh)

The `generate.sh` file:

1. Removes any outdated ZIP files previously generated for the project.
2. Copies the `connected-kiosk.json` file - this file is critical to the Notehub firmware running correctly.
3. Copies all the relevant files inside of the `web-app/` directory into a new directory named `resources/`.
   - This includes `index.htm`, `styles.css`, `data.js`, and a bunch of small image icons used to display current weather conditions and styled backgrounds inside of the `images/` folder and subfolders.
4. Runs the `package.sh` script (this script actually handles zipping together all the freshly copied files in the `resources/` folder).  
5. Copies the new `kiosk.zip` file inside of the `web-app/` folder.
6. Cleans up the (now unnecessary) extra `kiosk` files and `resources/` folder.

The `package.sh` file:

1. Deletes any old `kiosk.zip` files that might still be present in the project.
2. Creates a `metadata/` folder to separately store the copied `connected-kiosk.json` file data, which is pertinent to the Notehub firmware.
3. Builds a ZIP file named `kiosk.zip` from all the newly copied files and folders of `metadata/` and `resources/`.
4. Cleans up the `metadata/` folder afterwards.

### Create a Fresh ZIP of the Demo App

To package up the existing demo app, navigate into the `web-app/` directory:

```shell
cd web-app/
```

And run the following command:

```shell
./generate.sh
```

> **NOTE:** When running either script for the first time from the terminal, you may receive an error that you don't have permissions to execute the script. Instead of trying to run `sudo ./generate.sh` (which won't work), run `chmod +x generate.sh` or `chmod +x package.sh` - this will allow the project to execute the script, which is all the permissions it needs. Then run `./generate.sh` again in the terminal.

By the end of the script, there should be a new version of the `kiosk.zip` file inside of the `web-app/` folder.

## Upload the ZIP to a Cloud 

This new ZIP file for the kiosk created in the previous section needs to be uploaded to a repository where the Notehub cloud can access it via HTTP. For this example, an [Amazon Web Services S3 repo](https://aws.amazon.com/s3/) was used.

### Create New S3 Bucket

Inside of the AWS Console, navigate to the Amazon S3 bucket and click the **Create Bucket** button in the upper right hand corner of the screen.

**Name the bucket** 

Give the bucket an easily recognizable name.

![Name new AWS bucket](images/aws-create-bucket.png)

**Enable the bucket's ACLs**

Make the bucket's contents ownable by other AWS accounts.

![Select bucket's ownership option](images/aws-bucket-ownership.png)

**Make the bucket publicly accessible**

In order for Notehub to download the ZIP file from the bucket, the bucket will need to be publicly accessible.

Amazon warns this is a dangerous move, but uncheck the **Block all public access** check box and acknowledge the bucket and its contents could become public.

![Allow bucket to be accessed by the public](images/aws-public-bucket.png)

### Upload the ZIP to the S3 Bucket

Now that the bucket is created, click on its name in the list of buckets and upload the ZIP file to it by clicking the **Upload** button.

**Add files to the bucket**

On the Upload screen, click the **Add files** button and select the locally created `kiosk.zip` file.

![Upload the ZIP file to the bucket](images/aws-upload-zip.png)

**Grant public-read access in permissions**

Open the **Permissions** dropdown beneath where you selected the file, and select the **Grant public-read access** radio button, once again, Amazon will let you know this is risky, but acknowledge the risk because Notehub needs to be able to reach and download the contents of this bucket.

![Make the ZIP file accessible to the public in the bucket](images/aws-public-zip.png)

Then hit the **Upload** button at the end of the page.

If all goes well, you'll see an Upload Status page with a success message confirming the ZIP file was successfully uploaded. Don't close this tab yet though, you'll need some additional information from it shortly for Notehub.

![Success message when file uploaded successfully to bucket](images/aws-successful-upload.png)

## Create a Notehub Proxy Route Pointing Towards the AWS Bucket

At this point, you should have already created a Notehub project for this app from following along with the [`README.md` documentation](https://github.com/blues/app-accelerators/blob/main/27-cellular-connected-electronic-kiosk/README.md) in the main app-accelerators repo, if not, go ahead and do so now.

**Make a new proxy route in Notehub**

Once the Notehub project exists, inside of the project's **Routes** tab, click the **Create Route** button in the upper right hand corner of the screen, and select a **Proxy** route type.

![Choose to create a new proxy Notehub route](images/notehub-create-proxy.png)

**Get the URL for Notehub from the AWS ZIP file**

Go back to the AWS S3 bucket with the ZIP file and click on the file name to see its information. Here in the kiosk.zip page, copy the **Object URL** - this is what goes into the **URL** input of the Notehub route.

![Copy the Object URL from the AWS ZIP file's info](images/aws-bucket-url.png)

**Paste the URL into the Notehub route and give the route an alias**

Back over in the Notehub route, paste the AWS URL into the **URL** input and remove the `/kiosk.zip` from the end of the string. Give the route an alias - something like `kiosk`. This alias must match what's in the Notehub environment variable `kiosk_content`, and it's how the Python script will know which ZIP file to download if there are multiple ZIP files contained within the same S3 bucket.

![Paste the Object URL into the Notehub route and give it an alias](images/notehub-proxy.png)

## Make Sure the Notehub Fleet Environment Variables are Set

Once again, if you were following along with the [main `README.md` for this project](https://github.com/blues/app-accelerators/blob/main/27-cellular-connected-electronic-kiosk/README.md), you're aware that there are 4 Notehub environment variables needed for this application.

* `kiosk_content` - indicates the ZIP file to be downloaded from the Notehub proxy route.
* `kiosk_content_version` - a version number for the file specified by `kiosk_content`. If the content's updated, this number can be incremented to cause the Python script to download the new content.
* `kiosk_download_time` - the hour of the day for the Python script to check for updates to the `kiosk_content` env var. It can also be set to `now` to download as soon as new content is detected either by changing the `kiosk_content` variable or the `kiosk_content_version` number.
* `kiosk_data` - An optional JSON object that can be used to overwrite the contents of the `data.js` in the ZIP file. 

Keep in mind, that however the code in `data.js` is structured, the JSON in this `kiosk_data` environment variable must mimic it. For example, in the code, `data.js` looks like this:

```javascript
var data = {
  cities: ["Barcelona", "Madrid", "Dublin", "Tokyo"],
};
```

So for the `kiosk_data` Notehub environment variable, I created the following JSON object:

```json
{"cities": ["San Fernando", "Port of Spain", "Atlanta", "Shen Zhen"]}
```

When the project's Python script runs on the Raspberry Pi, it will read this variable from Notehub and prepend `var data=` to the front of the list, so it can sub in for the `data.js` file in the project.

Here's what this demo project's Notehub env vars look like all together.

![Notehub fleet env vars for the cellular-connected electronic kiosk](images/notehub-env-vars.png)

## Initiate the Download of the ZIP to the Raspberry Pi

Now it's time to download the ZIP file to the Raspberry Pi via the Notecard's Python script.

Following the instructions in the [`script/` directory's `README.md`](../scripts/README.md):

1. Clone this repo to the Raspberry Pi from GitHub, 
2. Ensure the Pi's version of Python is 3.8 or above, 
3. Open the repo with VS Code or the IDE of your choice, and 
4. Type the following command into a terminal at the root of the project substituting your own `data-dir`, `product`, and `route`.

```shell
python kiosk.py --data-dir ~/kiosk-data/ --product com.blues.cellular_XXXXXXXXX_XXXX --route kiosk 
```

This should initiate the download of the ZIP file from the AWS bucket via Notehub. Depending on the size of the ZIP, the download to the Pi may take a few minutes, but once it's finished, it will automatically open a Chromium web browser displaying the web app.

## Update the Notehub Env Vars and See the Changes Take Effect in the App

At this point, you may have noticed that the cities being displayed in the web app on the Pi are not the same ones contained within the `data.js` file in the ZIP. This is because the Notehub environment variable `kiosk_data` is overriding the list of cities contained within the `data.js` file. `kiosk_data` acts as a way of passing dynamic data to the web app without requiring a new ZIP file to be uploaded to the AWS bucket and downloaded by the Notecard.

To test this theory, go ahead and update the list of cities in the `kiosk_data` environment variable in Notehub to four different cities and save the change. 

If you're watching the still running Python script on the Pi's terminal, you should see it print out `Writing new data: {"cities": ...}` within several seconds. 

Switch back over to the Chromium tab displaying the web app, you should also see the list of cities being displayed update in a short amount of time without having to refresh the page as well.
