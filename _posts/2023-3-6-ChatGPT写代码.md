ChatGPT的写的代码，拿来可以直接用
调用ffmepg来批量转换视频格式

```python
import os
import subprocess
import json
import ffmpeg

def convert_to_hevc_qsv(input_path, output_path):
    """
    将视频转换为HEVC格式，编码器为hevc_qsv
    :param input_path: 输入文件路径
    :param output_path: 输出文件路径
    """
    # 调用ffmpeg进行转换
    stream = ffmpeg.input(input_path,hwaccel_output_format='qsv',vcodec='h264_qsv')
    stream = ffmpeg.output(stream, output_path,vcodec='hevc_qsv',acodec='copy',preset='veryslow',global_quality=25,look_ahead=1)
    ffmpeg.run(stream,overwrite_output=True)

def get_video_format(file_path):
    """
    使用ffprobe获取视频格式
    :param file_path: 文件路径
    :return: 视频格式
    """
    cmd = ['ffprobe', '-v', 'error', '-select_streams', 'v:0', '-show_entries', 'stream=codec_name', '-of', 'json', file_path]
    output = subprocess.check_output(cmd)
    output = output.decode('utf-8')
    data = json.loads(output)
    if 'streams' in data:
        codec_name = data['streams'][0]['codec_name']
        if codec_name == 'h264':
            return 'mp4'
        elif codec_name == 'hevc':
            return 'hevc'
    return None

def convert_videos_to_hevc_qsv(input_folder, output_folder, delete_input=False):
    """
    批量将非HEVC格式的mp4视频转换为HEVC格式，编码器为hevc_qsv，并删除源文件
    :param input_folder: 输入文件夹路径
    :param output_folder: 输出文件夹路径
    :param delete_input: 是否删除源文件，默认为False
    """
    # 遍历所有文件
    for root, dirs, files in os.walk(input_folder):
        for file in files:
            # 判断文件格式是否为mp4
            if os.path.splitext(file)[1].lower() != '.mp4':
                continue
            # 判断文件是否为要转换的视频格式
            input_path = os.path.join(root, file)
            if get_video_format(input_path) != 'hevc':
                # 构造输出文件路径
                output_path = os.path.join(output_folder, os.path.relpath(input_path, input_folder))
                output_path = os.path.splitext(output_path)[0] + ".hevc"
                os.makedirs(os.path.dirname(output_path), exist_ok=True)
                # 转换为HEVC格式
                try:
                    convert_to_hevc_qsv(input_path, output_path)
                except Exception as e:
                    print(f"Error converting file {input_path}: {str(e)}")
                    continue

                # 验证转换是否成功，如果成功则删除源文件
                if get_video_format(output_path) == 'hevc':
                    print(f"Successfully converted file {input_path}")
                    if delete_input:
                        os.remove(input_path)
                        print(f"Deleted input file {input_path}")
                else:
                    print(f"Failed to convert file {input_path}")

    print("Conversion completed!")

# 测试
input_folder = "path/to/input/folder"
output_folder = "path/to/output/folder"
convert_videos_to_hevc_qsv(input_folder, output_folder, delete_input=True)

```
