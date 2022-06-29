resource "aws_security_group" "allow_tls" {
  name        = "alllow_tls"
  description = "Allow TLS inbound traffic"

  //dynamic "ingress" {
  // for_each = [22,80,443,3306] ; var.ports (if you define in variable)
  // iterator = port
  // content{
  //   description      = "TLS from VPC"
  // from_port        = port.value
  // to_port          = port.value
  // protocol         = "tcp"
  // cidr_blocks      = ["0.0.0.0/0"]
  // }

  //}
  ingress {
    description = "TLS from VPC"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]

  }

  ingress {
    description = "TLS from VPC"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]

  }

  ingress {
    description = "TLS from VPC"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]

  }

  ingress {
    description = "TLS from VPC"
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]

  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }
}


# creating ssh key

resource "aws_key_pair" "deployer" {
  key_name   = "terraformm-key"
  public_key = "ssh-rsa AAAAAAAAAAAAAAAAAAAAAAFFFFFFFFFFF"
}
// public_key = file("${path.module}/id_rsa.pub")




resource "aws_instance" "web" {
  ami                    = "ami-04505e74c0741db8d"
  instance_type          = "t2.micro"
  vpc_security_group_ids = ["${aws_security_group.allow_tls.id}"]
  key_name               = aws_key_pair.deployer.key_name
  tags = {
    Name = "first-ttf-instance"
  }
  user_data = <<-EOF
#!/bin/bash
sudo apt update -y
sudo apt-get install nginx -y
sudo echo "Hi Vishal" >/var/www/html/index.nginx-debian.html
EOF

connection {
  type = "ssh"
  user = "ubuntu"
  private_key = file("${path.module}/id_rsa")
  host = "${self.public_ip}"
  }

provisioner "file" {
  source = "readme.md"
  destination = "/tmp/readme.md"
}

  provisioner "file" {
  content = "Vishal is a DevOps Engineer"
  destination = "/tmp/content.md"
 
  }

 provisioner "local-exec" {
   // working_dir = "/tmp"
   // on_failure = continue
   command = "echo ${self.public_ip} > /tmp/mypublic.txt"
 }

 provisioner "local-exec" {
   // on_failure = continue
   when = destroy
   // working_dir = "/tmp"
   command = "echo 'destroying'"
 }

 provisioner "remote-exec" {
   inline = ["ifconfig > /tmp/ifconfig.output",
                "echo 'hello gaurav'>/tmp/test.txt"
    ]
   
 }

 provisioner "remote-exec" {
   script = "./testscript.sh"
   
 }
 
}

provider "aws" {
  region     = "us-east-1"
  access_key = ""
  secret_key = ""
}


