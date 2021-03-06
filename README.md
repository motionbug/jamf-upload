# Jamf Pro upload scripts

- jamf_pkg_upload.py
- jamf_script_upload.py
- jamf_category_upload.py

## jamf_pkg_upload.py

Upload one or more packages to a Jamf Cloud Distribution Point or an SMB FileShare Distribution Point

    usage: jamf_pkg_upload.py [-h] [--replace] [--curl] [--direct] [--url URL]
                        [--user USER] [--password PASSWORD] [--share SHARE]
                        [--shareuser SHAREUSER] [--sharepass SHAREPASS]
                        [--category CATEGORY] [--timeout TIMEOUT]
                        [--chunksize CHUNKSIZE] [--prefs PREFS] [-v]
                        pkg [pkg ...]

### Positional arguments

    pkg                  Full path to the package(s) to upload

### Optional arguments

    -h, --help           show help message and exit
    --replace            overwrite an existing uploaded package (experimental)
    --curl               use curl instead of requests (experimental)
    --direct             use direct upload to JCDS (experimental, will not work
                         if JCDS is not primary distribution point)
    --chunksize CHUNKSIZE
                         set chunk size in megabytes (for direct method only)
    --url URL            the Jamf Pro Server URL
    --user USER          a user with the rights to upload a package
    --password PASSWORD  password of the user with the rights to upload a
                         package
    --timeout TIMEOUT    set timeout in seconds for HTTP request for problematic
                         packages
    --share SHARE        Path to an SMB FileShare Distribution Point, in the
                         form smb://server/mountpoint
    --shareuser SHAREUSER
                         a user with the rights to upload a package to the SMB
                         FileShare Distribution Point
    --sharepass SHAREPASS
                         password of the user with the rights to upload a
                         package to the SMB FileShare Distribution Point
    --category CATEGORY  a category to assign to the package (experimental)
    --prefs PREFS        full path to an AutoPkg prefs file containing JSS URL,
                         API_USERNAME and API_PASSWORD, for example an AutoPkg
                         preferences file which has been configured for use with
                         JSSImporter
                         (~/Library/Preferences/com.github.autopkg.plist) or a
                         separate plist anywhere (e.g.
                         ~/.com.company.jcds_upload.plist)
    -v, --verbose        print verbose output headers

Missing arguments (URL, username, password) will be asked for interactively if not already supplied.

### Examples

Here, we supply the JSS URL, API user and password, and the package to upload.

    ./jamf_pkg_upload.py --url https://myserver.jamfcloud.com --user jamf_admin --password myPasswordIsSecure /path/to/FoldingAtHome-7.5.1.pkg

Here, we point to the AutoPkg preferences file, whicb should contains the JSS URL, API user and password. We add verbosity and specify that the package should be replaced.

    ./jamf_pkg_upload.py --prefs ~/Library/Preferences/com.github.autopkg.plist -vv --replace ~/Library/AutoPkg/Cache/local.pkg.FoldingAtHome/FoldingAtHome-7.5.1.pkg

Here, we point to a custom plist, whicb should contains the JSS URL, API user and password. We specify that the package should be replaced, and supply a category to assign to the package object.

    ./jamf_pkg_upload.py --prefs ../credentials/custom.plist --category Applications --replace ~/Library/AutoPkg/Cache/local.pkg.FoldingAtHome/FoldingAtHome-7.5.1.pkg

Here, we point to the AutoPkg preferences file, whicb should contains the JSS URL, API user and password. We specify multiple packages that we wish to be uploaded in one run.

    ./jamf_pkg_upload.py --prefs ~/Library/Preferences/com.github.autopkg.plist \
    /path/to/FoldingAtHome-7.5.1.pkg \
    /path/to/AdoptOpenJDK11-11.0.6.pkg \
    /path/to/Firefox-77.0.1.pkg \
    ...

### Known issues in latest version

`jamf_pkg_upload.py` uses an undocumented API for uploading packages to Jamf Cloud. As far as I know, this is the same API used by the Jamf Admin app. It is also the same method used by JSSImporter.

The HTTP responses from this API are unpredictable. You may see a `504` response, which indicates a Gateway Timeout error, but the package may well be delivered anyway. This is true whether uploading a new package or overwriting an existing one.

As the HTTP response cannot be guaranteed, and package metadata such as category, manifest etc., must be supplied by a different API call after the package object has been uploaded, it can be unpredictable as to whether the package will be successfully uploaded. For this reason, please consider the `--category` option as experimental when uploading to Jamf Cloud.

The script also provides the `--direct` option, which uses a method resembling the way the GUI performs uploads. In our tests, this is more reliable at completing the package upload and providing the package ID as a response, which means the `--category` option should work more of the time.

### AutoPkg users

Users of AutoPkg can use the `JamfPackageUploader` processor to upload packages. It shares the functionality of this script, though will only upload one package per process. This can be run as a post-processor, e.g.:

    autopkg run FoldingAtHome.pkg --post com.github.grahampugh.recipes.postprocessors/JamfCloudPackageUploader

When running this processor, the `JSS_URL`, `API_USER` and `API_PASSWORD` preferences must be supplied in your AutoPkg preferences.

The processor could also be added to an override, or a new recipe could be made wth the `.pkg` recipe as its parent. As long as there is a `pkg_path` output from the `.pkg` recipe, no parameters would need to be supplied. If not, you could supply a value for the `pkg_path` key as an argument to the processor. This would allow you to use the `.download` recipe as a parent if the download is a valid `pkg`.

Please don't use the `.jss` suffix for such a recipe if you publish it, as that would confuse the recipe with JSSImporter recipes. I suggest `.jamf-upload.recipe`.

## jamf_script_upload.py

Upload one or more scripts to a Jamf Pro server using the API.

    usage: jamf_script_upload.py [-h] [--replace] [--url URL] [--user USER]
                                [--password PASSWORD] [--category CATEGORY]
                                [--priority PRIORITY]
                                [--osrequirements OSREQUIREMENTS] [--info INFO]
                                [--notes NOTES] [--parameter4 PARAMETER4]
                                [--parameter5 PARAMETER5]
                                [--parameter6 PARAMETER6]
                                [--parameter7 PARAMETER7]
                                [--parameter8 PARAMETER8]
                                [--parameter9 PARAMETER9]
                                [--parameter10 PARAMETER10]
                                [--parameter11 PARAMETER11] [--prefs PREFS] [-v]
                                script [script ...]

    positional arguments:
        script                Full path to the script(s) to upload

    optional arguments:
        -h, --help              show this help message and exit
        --replace               overwrite an existing uploaded script
        --url URL               the Jamf Pro Server URL
        --user USER             a user with the rights to upload a script
        --password PASSWORD     password of the user with the rights to upload a
                                script
        --category CATEGORY     a category to assign to the script(s)
        --priority PRIORITY     priority to assign to the script(s) - BEFORE or AFTER
        --osrequirements OSREQUIREMENTS
                                a value to assign to the OS requirements field of the
                                script(s)
        --info INFO             information to assign to the script(s)
        --notes NOTES           notes to assign to the script(s)
        --parameter4 PARAMETER4
                                a value to assign to parameter4 of the script(s)
        --parameter5 PARAMETER5
                                a value to assign to parameter5 of the script(s)
        --parameter6 PARAMETER6
                                a value to assign to parameter6 of the script(s)
        --parameter7 PARAMETER7
                                a value to assign to parameter7 of the script(s)
        --parameter8 PARAMETER8
                                a value to assign to parameter8 of the script(s)
        --parameter9 PARAMETER9
                                a value to assign to parameter9 of the script(s)
        --parameter10 PARAMETER10
                                a value to assign to parameter10 of the script(s)
        --parameter11 PARAMETER11
                                a value to assign to parameter11 of the script(s)
        --prefs PREFS           full path to an AutoPkg prefs file containing JSS URL,
                                API_USERNAME and API_PASSWORD, for example an AutoPkg
                                preferences file which has been configured for use
                                with JSSImporter
                                (~/Library/Preferences/com.github.autopkg.plist) or a
                                separate plist anywhere (e.g.
                                ~/.com.company.jcds_upload.plist)
        -v, --verbose           print verbose output headers

## jamf_category_upload.py

Create or update one or more categories on a Jamf Pro server using the API.

    usage: jamf_category_upload.py [-h] [--priority PRIORITY] [--url URL]
                                [--user USER] [--password PASSWORD]
                                [--prefs PREFS] [-v]
                                category [category ...]

    positional arguments:
    category             Category to create

    optional arguments:
    -h, --help           show this help message and exit
    --priority PRIORITY  Category priority. Default value is 10
    --url URL            the Jamf Pro Server URL
    --user USER          a user with the rights to create a category
    --password PASSWORD  password of the user with the rights to create a
                         category
    --prefs PREFS        full path to an AutoPkg prefs file containing JSS URL,
                         API_USERNAME and API_PASSWORD, for example an AutoPkg
                         preferences file which has been configured for use with
                         JSSImporter
                         (~/Library/Preferences/com.github.autopkg.plist) or a
                         separate plist anywhere (e.g.
                         ~/.com.company.jcds_upload.plist)
    -v, --verbose        print verbose output headers
