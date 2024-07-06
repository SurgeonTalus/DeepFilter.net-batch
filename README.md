# DeepFilter.net-batch
batch deepfilter on Mac

FUNGERER ALENE BATCH:

#!/bin/bash

# Directory containing the videos to process
input_dir="/Volumes/500GB ExFAT/Begravelse/Processed/Test"

# Loop through each video file in the directory
for video_file in "$input_dir"/*.MOV; do
    if [ -f "$video_file" ]; then
        video_name=$(basename "$video_file")
        video_name_no_ext="${video_name%.*}"

        # Step 1: Extract audio from the MOV file and save it as WAV
        ffmpeg -i "$video_file" \
               -vn -acodec pcm_s16le -ar 48000 -ac 2 \
               "${input_dir}/${video_name_no_ext}.wav" \
        &&

        # Step 2: Process the extracted audio using deepFilter
        deepFilter "${input_dir}/${video_name_no_ext}.wav" \
                   --output-dir "${input_dir}" \
        &&

        # Step 3: Check if _DeepFilterNet3.wav exists before proceeding
        while [ ! -f "${input_dir}/${video_name_no_ext}_DeepFilterNet3.wav" ]; do
            echo "Waiting for ${video_name_no_ext}_DeepFilterNet3.wav to be created..."
            sleep 1
        done

        # Step 4: Combine original video, processed audio, and original audio with loudness adjustment
        ffmpeg -i "$video_file" \
               -i "${input_dir}/${video_name_no_ext}_DeepFilterNet3.wav" \
               -i "${input_dir}/${video_name_no_ext}.wav" \
               -filter_complex "[1:a]volume=1[a1];[2:a]volume=0.05[a2];[a1][a2]amix=inputs=2:duration=first[outa]" \
               -map 0:v:0 -map "[outa]" \
               -c:v copy \
               "${input_dir}/${video_name_no_ext}_filtered.MOV"

        echo "Processed $video_name"
    fi
done
