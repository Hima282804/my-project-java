name: CI/CD Integration & Deployment

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout source
      uses: actions/checkout@v3

    - name: Set up Node
      uses: actions/setup-node@v3
      with:
        node-version: '18'

    - name: Install and test
      run: |
        npm ci
        npm test

    - name: Build frontend
      run: |
        cd frontend
        npm run build

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}

    - name: Upload frontend build to S3
      run: |
        aws s3 sync frontend/build/ s3://${{ secrets.S3_BUCKET_NAME }} --delete

    - name: Deploy backend to EC2
      uses: appleboy/ssh-action@v0.1.10
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USERNAME }}
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          cd /home/${{ secrets.EC2_USERNAME }}/my-backend
          git pull origin main
          npm install
          pm2 restart backend || pm2 start server.js --name backend
