ChatGPT的写的代码，拿来可以直接用
调用ffmepg来批量转换视频格式

```python
import os
import subprocess
import json

input_folder = r"C:\APP\Test"
output_folder = r"C:\APP\mp4"

# 遍历文件夹下的所有文件，包括子文件夹
for root, dirs, files in os.walk(input_folder):
    for file in files:
        # 判断文件格式是否为mp4
        if file.endswith(".mp4"):
            input_path = os.path.join(root, file)

            # 使用ffprobe获取视频流信息
            cmd = f'ffprobe -v error -select_streams v:0 -show_entries stream=codec_name -of json "{input_path}"'
            process = subprocess.Popen(cmd, stdout=subprocess.PIPE, stderr=subprocess.STDOUT, shell=True)
            output, _ = process.communicate()
            codec = json.loads(output)["streams"][0]["codec_name"]

            # 判断视频编码是否为hevc
            if codec != "hevc":
                output_path = os.path.join(output_folder, os.path.relpath(input_path, input_folder))
                output_path = os.path.splitext(output_path)[0] + ".mp4"
                os.makedirs(os.path.dirname(output_path), exist_ok=True)
                # 使用ffmpeg进行格式转换
                cmd = f'ffmpeg -y -init_hw_device qsv:hw,child_device_type=d3d11va -hwaccel_output_format qsv -loglevel verbose -i "{input_path}" -c:v hevc_qsv -c:a copy -preset veryslow -global_quality 25 -look_ahead 1 -pix_fmt nv12 "{output_path}"'
                if subprocess.run(cmd,shell=True).returncode==0:
                    os.replace(output_path, input_path)
                    print(f"文件 {input_path} 转换成功，已替换源文件")
                else:
                    print(f"文件 {input_path} 转换失败，请检查输出信息")
```
