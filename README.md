aws-eks
=======

## Usage
```bash
export AWS_REGION=ap-southeast-1
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=

atmos vendor pull

atmos workflow init -f main
atmos workflow plan -f main
atmos workflow apply -f main
atmos workflow destroy -f main
```
