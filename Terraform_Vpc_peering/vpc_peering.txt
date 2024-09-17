# Create VPC 1
resource "aws_vpc" "vpc1" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "VPC1"
  }
}

# Create Subnet for VPC 1
resource "aws_subnet" "subnet1" {
  vpc_id                  = aws_vpc.vpc1.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true
  tags = {
    Name = "Subnet1"
  }
}

# Create Internet Gateway for VPC 1
resource "aws_internet_gateway" "igw1" {
  vpc_id = aws_vpc.vpc1.id
  tags = {
    Name = "IGW1"
  }
}

# Create Route Table for VPC 1 and associate with the subnet
resource "aws_route_table" "rt1" {
  vpc_id = aws_vpc.vpc1.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw1.id
  }
}

resource "aws_route_table_association" "rta1" {
  subnet_id      = aws_subnet.subnet1.id
  route_table_id = aws_route_table.rt1.id
}

# Create Security Group for VPC 1
resource "aws_security_group" "sg1" {
  name   = "vpc1-sg"
  vpc_id = aws_vpc.vpc1.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Create EC2 Instance in VPC 1
resource "aws_instance" "instance1" {
  ami                         = "ami-0e86e20dae9224db8" # Replace with a valid AMI ID
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.subnet1.id
  key_name                    = "Terra"  # Replace with your AWS key pair name
  vpc_security_group_ids      = [aws_security_group.sg1.id]
  associate_public_ip_address = true

  tags = {
    Name = "Instance1"
  }
}

# Create VPC 2
resource "aws_vpc" "vpc2" {
  cidr_block = "10.1.0.0/16"
  tags = {
    Name = "VPC2"
  }
}

# Create Subnet for VPC 2
resource "aws_subnet" "subnet2" {
  vpc_id                  = aws_vpc.vpc2.id
  cidr_block              = "10.1.1.0/24"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true
  tags = {
    Name = "Subnet2"
  }
}

# Create Internet Gateway for VPC 2
resource "aws_internet_gateway" "igw2" {
  vpc_id = aws_vpc.vpc2.id
  tags = {
    Name = "IGW2"
  }
}

# Create Route Table for VPC 2 and associate with the subnet
resource "aws_route_table" "rt2" {
  vpc_id = aws_vpc.vpc2.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw2.id
  }
}

resource "aws_route_table_association" "rta2" {
  subnet_id      = aws_subnet.subnet2.id
  route_table_id = aws_route_table.rt2.id
}

# Create Security Group for VPC 2
resource "aws_security_group" "sg2" {
  name   = "vpc2-sg"
  vpc_id = aws_vpc.vpc2.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Create EC2 Instance in VPC 2
resource "aws_instance" "instance2" {
  ami                         = "ami-0e86e20dae9224db8" # Replace with a valid AMI ID
  instance_type               = "t2.micro"
  subnet_id                   = aws_subnet.subnet2.id
  key_name                    = "Terra"  # Replace with your AWS key pair name
  vpc_security_group_ids      = [aws_security_group.sg2.id]
  associate_public_ip_address = true

  tags = {
    Name = "Instance2"
  }
}

# Create VPC Peering Connection
resource "aws_vpc_peering_connection" "peer" {
  vpc_id        = aws_vpc.vpc1.id
  peer_vpc_id   = aws_vpc.vpc2.id
  auto_accept   = true

  tags = {
    Name = "VPC1-to-VPC2-Peering"
  }
}

# Update Route Tables to include Peering Connection
resource "aws_route" "route_to_vpc2" {
  route_table_id         = aws_route_table.rt1.id
  destination_cidr_block = aws_vpc.vpc2.cidr_block
  vpc_peering_connection_id = aws_vpc_peering_connection.peer.id
}

resource "aws_route" "route_to_vpc1" {
  route_table_id         = aws_route_table.rt2.id
  destination_cidr_block = aws_vpc.vpc1.cidr_block
  vpc_peering_connection_id = aws_vpc_peering_connection.peer.id
}
