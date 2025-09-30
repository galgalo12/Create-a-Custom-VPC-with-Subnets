create-vpc-cf-1-NGW.yaml




      # VPC Configuration
      
      ## VPC
      - **Name:** `vpc`  
      - **IPv4 CIDR Block:** `10.0.0.0/16`
      
      ## Subnets
      | Type            | Name       | Availability Zone | IPv4 CIDR Block | Auto-Assign Public IP |
      |-----------------|------------|-------------------|-----------------|-----------------------|
      | Public Subnet 1 | Public-1A  | us-east-1a        | 10.0.1.0/24     | Enabled |
      | Public Subnet 2 | Public-1B  | us-east-1b        | 10.0.2.0/24     | Enabled |
      | Private Subnet 1| Private-1A | us-east-1a        | 10.0.3.0/24     | Disabled |
      | Private Subnet 2| Private-1B | us-east-1b        | 10.0.4.0/24     | Disabled |
      
      ## Route Table
      - **Name:** `Private-RT`
      - **ID:** `0000`
      - **VPC:** `MyVPC`
      - **Subnet Associations:** `Private-1A`, `Private-1B`
      
      ## Internet Gateway
      - **Name:** `MyIGW`
      - **Attached to VPC:** `MyVPC`
      
      ---
      
      ### Testing Steps
      1. **Ping/SSH Test**  
         - Launch an EC2 instance in **Public-1A** or **Public-1B** with a public IP.
         - SSH in from your workstation to verify Internet access.
      
      2. **Private Subnet Access**  
         - Launch an EC2 instance in **Private-1A** or **Private-1B** (no public IP).
         - SSH to it from the public instance using its private IP to confirm internal routing.
      
      3. **Security Group/Network ACL Checks**  
         - Ensure inbound/outbound rules allow SSH/ICMP as needed.


Description:  This template deploys a VPC, with a pair of public and private subnets spread
  across two Availability Zones. It deploys an internet gateway, with a default
  route on the public subnets. It deploys a pair of NAT gateways (one in each AZ),
  and default routes for them in the private subnets.

  Parameters:
    EnvironmentName:
      Description: An environment name that is prefixed to resource names
      Type: String
  
    VpcCIDR:
      Description: Please enter the IP range (CIDR notation) for this VPC
      Type: String
      Default: 10.8.0.0/16
  
    PublicSubnet1CIDR:
      Description: Please enter the IP range (CIDR notation) for the public subnet in the first Availability Zone
      Type: String
      Default: 10.8.10.0/24
  
    PublicSubnet2CIDR:
      Description: Please enter the IP range (CIDR notation) for the public subnet in the second Availability Zone
      Type: String
      Default: 10.8.11.0/24
  
    PrivateSubnet1CIDR:
      Description: Please enter the IP range (CIDR notation) for the private subnet in the first Availability Zone
      Type: String
      Default: 10.8.20.0/24
  
    PrivateSubnet2CIDR:
      Description: Please enter the IP range (CIDR notation) for the private subnet in the second Availability Zone
      Type: String
      Default: 10.8.21.0/24
  
  Resources:
    VPC:
      Type: AWS::EC2::VPC
      Properties:
        CidrBlock: !Ref VpcCIDR
        EnableDnsSupport: true
        EnableDnsHostnames: true
        Tags:
          - Key: Name
            Value: !Ref EnvironmentName
  
    InternetGateway:
      Type: AWS::EC2::InternetGateway
      Properties:
        Tags:
          - Key: Name
            Value: !Ref EnvironmentName
  
    InternetGatewayAttachment:
      Type: AWS::EC2::VPCGatewayAttachment
      Properties:
        InternetGatewayId: !Ref InternetGateway
        VpcId: !Ref VPC
  
    PublicSubnet1:
      Type: AWS::EC2::Subnet
      Properties:
        VpcId: !Ref VPC
        AvailabilityZone: !Select [ 0, !GetAZs '' ]
        CidrBlock: !Ref PublicSubnet1CIDR
        MapPublicIpOnLaunch: true
        Tags:
          - Key: Name
            Value: !Sub ${EnvironmentName} Public Subnet (AZ1)
  
    PublicSubnet2:
      Type: AWS::EC2::Subnet
      Properties:
        VpcId: !Ref VPC
        AvailabilityZone: !Select [ 1, !GetAZs  '' ]
        CidrBlock: !Ref PublicSubnet2CIDR
        MapPublicIpOnLaunch: true
        Tags:
          - Key: Name
            Value: !Sub ${EnvironmentName} Public Subnet (AZ2)
  
    PrivateSubnet1:
      Type: AWS::EC2::Subnet
      Properties:
        VpcId: !Ref VPC
        AvailabilityZone: !Select [ 0, !GetAZs  '' ]
        CidrBlock: !Ref PrivateSubnet1CIDR
        MapPublicIpOnLaunch: false
        Tags:
          - Key: Name
            Value: !Sub ${EnvironmentName} Private Subnet (AZ1)
  
    PrivateSubnet2:
      Type: AWS::EC2::Subnet
      Properties:
        VpcId: !Ref VPC
        AvailabilityZone: !Select [ 1, !GetAZs  '' ]
        CidrBlock: !Ref PrivateSubnet2CIDR
        MapPublicIpOnLaunch: false
        Tags:
          - Key: Name
            Value: !Sub ${EnvironmentName} Private Subnet (AZ2)
  
    NatGateway1EIP:
      Type: AWS::EC2::EIP
      DependsOn: InternetGatewayAttachment
      Properties:
        Domain: vpc
  
    NatGateway2EIP:
      Type: AWS::EC2::EIP
      DependsOn: InternetGatewayAttachment
      Properties:
        Domain: vpc
  
    NatGateway1:
      Type: AWS::EC2::NatGateway
      Properties:
        AllocationId: !GetAtt NatGateway1EIP.AllocationId
        SubnetId: !Ref PublicSubnet1
  
    NatGateway2:
      Type: AWS::EC2::NatGateway
      Properties:
        AllocationId: !GetAtt NatGateway2EIP.AllocationId
        SubnetId: !Ref PublicSubnet2
  
    PublicRouteTable:
      Type: AWS::EC2::RouteTable
      Properties:
        VpcId: !Ref VPC
        Tags:
          - Key: Name
            Value: !Sub ${EnvironmentName} Public Routes
  
    DefaultPublicRoute:
      Type: AWS::EC2::Route
      DependsOn: InternetGatewayAttachment
      Properties:
        RouteTableId: !Ref PublicRouteTable
        DestinationCidrBlock: 0.0.0.0/0
        GatewayId: !Ref InternetGateway
  
    PublicSubnet1RouteTableAssociation:
      Type: AWS::EC2::SubnetRouteTableAssociation
      Properties:
        RouteTableId: !Ref PublicRouteTable
        SubnetId: !Ref PublicSubnet1
  
    PublicSubnet2RouteTableAssociation:
      Type: AWS::EC2::SubnetRouteTableAssociation
      Properties:
        RouteTableId: !Ref PublicRouteTable
        SubnetId: !Ref PublicSubnet2
  
  
    PrivateRouteTable1:
      Type: AWS::EC2::RouteTable
      Properties:
        VpcId: !Ref VPC
        Tags:
          - Key: Name
            Value: !Sub ${EnvironmentName} Private Routes (AZ1)
  
    DefaultPrivateRoute1:
      Type: AWS::EC2::Route
      Properties:
        RouteTableId: !Ref PrivateRouteTable1
        DestinationCidrBlock: 0.0.0.0/0
        NatGatewayId: !Ref NatGateway1
  
    PrivateSubnet1RouteTableAssociation:
      Type: AWS::EC2::SubnetRouteTableAssociation
      Properties:
        RouteTableId: !Ref PrivateRouteTable1
        SubnetId: !Ref PrivateSubnet1
  
    PrivateRouteTable2:
      Type: AWS::EC2::RouteTable
      Properties:
        VpcId: !Ref VPC
        Tags:
          - Key: Name
            Value: !Sub ${EnvironmentName} Private Routes (AZ2)
  
    DefaultPrivateRoute2:
      Type: AWS::EC2::Route
      Properties:
        RouteTableId: !Ref PrivateRouteTable2
        DestinationCidrBlock: 0.0.0.0/0
        NatGatewayId: !Ref NatGateway2
  
    PrivateSubnet2RouteTableAssociation:
      Type: AWS::EC2::SubnetRouteTableAssociation
      Properties:
        RouteTableId: !Ref PrivateRouteTable2
        SubnetId: !Ref PrivateSubnet2
  
    NoIngressSecurityGroup:
      Type: AWS::EC2::SecurityGroup
      Properties:
        GroupName: "no-ingress-sg"
        GroupDescription: "Security group with no ingress rule"
        VpcId: !Ref VPC
  
  Outputs:
    VPC:
      Description: A reference to the created VPC
      Value: !Ref VPC
  
    PublicSubnets:
      Description: A list of the public subnets
      Value: !Join [ ",", [ !Ref PublicSubnet1, !Ref PublicSubnet2 ]]
  
    PrivateSubnets:
      Description: A list of the private subnets
      Value: !Join [ ",", [ !Ref PrivateSubnet1, !Ref PrivateSubnet2 ]]
  
    PublicSubnet1:
      Description: A reference to the public subnet in the 1st Availability Zone
      Value: !Ref PublicSubnet1
  
    PublicSubnet2:
      Description: A reference to the public subnet in the 2nd Availability Zone
      Value: !Ref PublicSubnet2
  
    PrivateSubnet1:
      Description: A reference to the private subnet in the 1st Availability Zone
      Value: !Ref PrivateSubnet1
  
    PrivateSubnet2:
      Description: A reference to the private subnet in the 2nd Availability Zone
      Value: !Ref PrivateSubnet2
  
    NoIngressSecurityGroup:
      Description: Security group with no ingress rule
      Value: !Ref NoIngressSecurityGroup
