resource "aws_instance" "ubuntu_ec2" {
    count = 3
    ami = "ami-03cc8375791cb8bcf"
    instance_type = "t2.micro"
    key_name = "terra"
    tags = {
      Name = "ubuntuServer-${count.index + 1}"
    }    
}
