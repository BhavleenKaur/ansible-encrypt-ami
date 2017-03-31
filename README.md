# ansible-encrypt-ami

This role creates an encrypted copy of an EC2 AMI. Subsequent instance EBS volumes created from the encrypted AMI and any snapshots will automatically be encrypted.

https://aws.amazon.com/blogs/aws/new-encrypted-ebs-boot-volumes/

All tags on the source AMI are replicated to the resulting encrypted copy with the addition of an additional tag: `Encrypted: true`.

## Source AMI's

AMI's from the EC2 marketplace with product codes cannot be copied. This is a limitation of the Amazon API. This includes official Canonical and CentOS images for example. Ubuntu's [CloudImage AMI's](https://cloud-images.ubuntu.com/) are not in the official marketplace do not have product codes so they can be copied.

For those AMI's that cannot be copied one workaround is to create an AMI via Packer and then copy the resulting AMI to an encrypted AMI.

## Usage

Create a copy from a specific AMI id:

```yaml
- role: encrypt-ami
  encrypt_ami_source_ami_id: ami-000000000
```

Create a copy from an AMI matching tags:

```yaml
- role: encrypt-ami
  encrypt_ami_source_search_tags:
    Name: "{{ base_ami_name }}"
    Type: base
    Source: reactiveops
```

The default will only search for AMIs in 'self' / Account space. That can be changed by providing the owner.
For example, to create a copy from a Cononical / Ubuntu AMI:

```yaml
- role: encrypt-ami
  encrypt_ami_source_ami_search_owner: 099720109477
  encrypt_ami_source_ami_id: ami-af22d9b9
```

## Tests

See [tests/README.md](tests/README.md).
