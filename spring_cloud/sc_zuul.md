# Zuul

## 最佳实践

### 上传文件，经过Zuul，中文文件名乱码解决办法

假设正常的上传文件的URL地址为`localhost:5000/oss/file/upload`，那么如果中文文件名乱码，那么可以通过将该URL路径修改为`localhost:5000/zuul/oss/file/upload`。