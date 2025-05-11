```bash
aws s3api create-bucket --bucket meu-bucket-eks --region us-east-1 && \
OIDC_URL=$(aws eks describe-cluster --name meu-cluster --query "cluster.identity.oidc.issuer" --output text) && \
cat > trust-policy.json <<EOF
{
 "Version": "2012-10-17",
 "Statement": [{
   "Effect": "Allow",
   "Principal": {
     "Federated": "arn:aws:iam::<id>:oidc-provider/${OIDC_URL#https://}"
   },
   "Action": "sts:AssumeRoleWithWebIdentity",
   "Condition": {
     "StringEquals": {
       "${OIDC_URL#https://}:sub": "system:serviceaccount:default:s3-access-sa"
     }
   }
 }]
}
EOF
&& \
aws iam create-role --role-name EKS-S3-Access-Role --assume-role-policy-document file://trust-policy.json && \
aws iam attach-role-policy --role-name EKS-S3-Access-Role --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess && \
eksctl create iamserviceaccount --name s3-access-sa --namespace default --cluster meu-cluster --attach-role-arn arn:aws:iam::<id>:role/EKS-S3-Access-Role --approve
