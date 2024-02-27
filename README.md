- 👋 Hi, I’m @Mixxx1223
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Mixxx1223/Mixxx1223 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

variable "availability_zone" {
  default = "us-east-2a"
}
variable "vpc_id" {
  default = "vpc-d9a68bb1"
}

variable "subnet_count" {
  default = "1"
}

data "aws_vpc" "env" {
  id = "${var.vpc_id}"
}

data "external" "net" {
  program = ["/atf/bin/atf-subnet-reservation-alloc"]

  query = {
    subnet_count = "${var.subnet_count}"
    vpc_id = "${var.vpc_id}"
  }
}

resource "aws_subnet" "net" {
  count             = "${var.subnet_count}"
  vpc_id            = "${data.aws_vpc.env.id}"
  availability_zone = "${var.availability_zone}"
  cidr_block        = "${cidrsubnet(data.aws_vpc.env.cidr_block, 8, element(split(",",data.external.net.result.subnets),count.index))}"
  map_public_ip_on_launch = "true"

  provisioner "local-exec" {
    when = "destroy"
    command = "/atf/bin/atf-subnet-reservation-free ${self.cidr_block}"
    on_failure = "continue"
  }
}
