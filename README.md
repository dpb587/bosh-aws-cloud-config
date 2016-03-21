# bosh-aws-cloud-config

Some snippets to help compose a [BOSH][1] [cloud-config][2] on [AWS][3].

    $ ( cat azs/$AWS_DEFAULT_REGION.yml compilation.yml disk_types.yml vm_types.yml \
      ; generator/networks-vpc vpc-a1b2c3d4 \
      ) > cloud-config.yml
    $ vim cloud-config.yml
    $ bosh update cloud-config cloud-config.yml


## Availability Zones

Statically listed in the [`azs`](./azs/) directory. The name is the letter of the availability zone (e.g. `e`).


## Disk Types

Statically listed in the [`disk_types.yml`](./disk_types.yml) file. The name is the disk type and size in GBs, separated by underscore (e.g. `gp2_128`). General Purpose SSD (`gp2`) and Magnetic (`standard`) are the only two types listed - add `io1` types manually. Sizes start at 4 GB and double until 2 TB.


## VM Types

Statically listed in the [`vm_types.yml`](./vm_types.yml) file. The name is the instance type, with an underscore instead of a period (e.g. `c4_xlarge`). Instance types which do not offer instance storage will have a 16 GB EBS ephemeral disk attached.


## Networks

If your subnets are tagged with `Name`s, you can use the `generator/networks-vpc` script, passing it your VPC ID. Subnets with the same name will be collected into a single network, respecting availability zones. Once generated, you might want to update the `cloud_properties`, `dns`, `reserved`, and `static` keys. An `eip` network for Elastic IPs is also included. Note: this does not handle subnets smaller than `/24`.

    $ ./generator/networks-vpc vpc-a1b2c3d4
    networks:
      - { name: "eip", type: "vip" }
      - name: "private"
        subnets:
          - range: "10.0.0.0/20"
            az: "a"
            gateway: "10.0.0.1"
            reserved: [ "10.0.0.2", "10.0.0.3" ]
            dns: [ "169.254.169.253" ]
            cloud_properties: { subnet: "subnet-b1c2d3e4" }
          - range: "10.0.128.0/20"
            az: "b"
            gateway: "10.0.128.1"
            reserved: [ "10.0.128.2", "10.0.128.3" ]
            dns: [ "169.254.169.253" ]
            cloud_properties: { subnet: "subnet-c1d2e3f4" }


## Requirements

Unless you are copying from statically generated files, you will want [aws][6], [json2yaml][4], and [jq][5] accessible from your `$PATH`. Also make sure `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` are set.


## Regenerate

Sometimes AWS adds new regions and availability zones...

    $ generator/azs
    $ git commit -m 'update azs' azs/*

Sometimes AWS adds new instance types...

    $ vim generator/instance_types.json
    $ generator/vm_types
    $ git commit -m 'update vm_types' generator/instance_types.json vm_types.yml

Sometimes AWS adds new disk types...

    $ vim generator/disk_types
    $ generator/disk_types
    $ git commit -m 'update disk_types' disk_types.yml generator/disk_types


 [1]: https://bosh.io/
 [2]: https://bosh.io/docs/cloud-config.html
 [3]: https://aws.amazon.com/
 [4]: http://jsontoyaml.com/#bash
 [5]: https://stedolan.github.io/jq/
 [6]: https://aws.amazon.com/cli/
