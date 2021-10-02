# tvf or, terraform-version-freezer

**IMPORTANT**: This script requires a single file containing the whole terraform versions definition. It cannot search for it into your folder, nor it can extract it from a `.tf` file which includes other code than the versions resource.

This script will ingest a `versions.tf` file and will spit out a json file with all versions updated to the latest **patch** release available.

That is, if `hashicorp/aws` provider is set to be `~> 1.2.0`, then on `terraform init` you'll download the latest patch version, which could be `1.2.3`. When you're ready to release, then you'll likely want to fix that version with an `= 1.2.3`. The chances a version `1.2.4` could break the system are low, but it's anyway a good practice to fix those, along the terraform binary version.

**Optionally** (by setting the `-u` or `--update` switch) it will update the versions file with the latest versions available. *This works only with the `--json` switch*, it cannot read and write in hcl. This script is using the `hcl2` module (https://github.com/amplify-education/python-hcl2) which isn't currently allowing to convert back json to hcl, so we can only write json basically.

In this case, if the version of the `hashicorp/aws` provider into the versions file is set to be `~> 1.2.0` but the latest version online is `1.5.0`, then the script will update the file with `~> 1.5.0`.

## Install

Install the requirements as usual with `pip3 install -r requirements.txt`.

## Usage

Provided a `versions.tf` file in hcl (the original terraform file format) like in this example

```
terraform {
  required_version = ">= 0.15.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.45.0"
    }
  }
}
```

Once the script will run

```
tvf -f versions.tf
```

You'll get the current version based on the restriction, and the latest real version, ie

```
{
  "terraform": [
    {
      "required_version": "= 0.15.5",
      "required_providers": [
        {
          "aws": {
            "source": "hashicorp/aws",
            "version": "= 3.60.0"
          }
        }
      ]
    }
  ]
}

```


By adding `-w`, the script will *delete* the original `versions.tf` file and will write a new `versions.tf.json` file instead, with the versions set with `=` to the latest patch. Terraform works natively with json files, they just need to have the `.json` extension.

Once converted, the script will still work with the new json format, just target the new file and use `--json`


To do:
- inplace update to the real last version keeping the tilde operator (`~>`) instead of equal. This would help to just update the versions file instead of fixing it for release.


