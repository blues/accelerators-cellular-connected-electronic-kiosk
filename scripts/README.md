# Cellular-Connected Electronic Kiosk Scripts

The Cellular-Connected Electronic Kiosk's software script is built using Python. To run the scripts on your Raspberry Pi, make sure your Pi meets the following requirements.

## Dependencies

1. To run the kiosk software, you'll need at least Python 3.8. 

> The default Python version that comes with the OS should be sufficient, but you can check the version at the command line with `python --version`. 

2. Download this repo locally from GitHub to the Raspberry Pi.
3. You will also need to install a couple of Python packages. 
4. From the root of the repo, navigate to the `scripts/` directory.

```shell
cd accelerators-cellular-connected-electronic-kiosk/scripts/
```
4. Run the following commands to install the dependencies.
   
```shell
pip install -r requirements.txt
```

## Running

`kiosk.py` is a Python script that runs the application. With `python kiosk.py --help`, you'll see this help message:

```shell
usage: kiosk.py [-h] [--data-dir DATA_DIR] --product PRODUCT --route ROUTE
Run the Notecard-based kiosk app.
optional arguments:
  -h, --help           show this help message and exit
  --data-dir DATA_DIR  Root directory for downloaded content and other files created by this script.
  --product PRODUCT    ProductUID for the Notehub project.
  --route ROUTE        Alias for a Proxy Route in Notehub that will be used to download content.
```

- `--data-dir` is optional; if you don't specify it, the default data directory will be `~/kiosk-data/`.
- `--product` should be your [Notehub project's ProductUID](https://dev.blues.io/notehub/notehub-walkthrough/#finding-a-productuid).
- `--route` should be the `alias` of the proxy route you created.

After launching the script with `python kiosk.py --data-dir <path to your data directory> --product <ProductUID> --route <route alias>`, it will begin downloading the ZIP file via the proxy route. Once downloaded, it will unzip the file and open `resources/index.htm` in a browser window.