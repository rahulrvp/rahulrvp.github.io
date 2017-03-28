---
layout: post
title:  "Android: PCM file to AVI File"
date:   2017-03-28 13:12:55 +0530
categories: jekyll update
---
Here is a sample code that converts a `PCM` file into playable `AVI` file.

The `PCM` file I had was created using [AudioRecord][audio_record] class with a sample rate of 16000Hz.

This [sample code][voice_recorder] is the inspiration for my audio recorder that created my source `PCM` file.


```java
private File getAviFileFromRawPCM(File rawFile, File outputFile) throws IOException {
    int SAMPLE_RATE = 16000;

    byte[] rawData = new byte[(int) rawFile.length()];
    DataInputStream input = null;
    try {
        input = new DataInputStream(new FileInputStream(rawFile));
        input.read(rawData);
    } finally {
        if (input != null) {
            input.close();
        }
    }

    DataOutputStream output = null;
    try {
        output = new DataOutputStream(new FileOutputStream(outputFile));
        // WAVE header
        // see http://ccrma.stanford.edu/courses/422/projects/WaveFormat/
        writeString(output, "RIFF"); // chunk id
        writeInt(output, 36 + rawData.length); // chunk size
        writeString(output, "WAVE"); // format
        writeString(output, "fmt "); // subchunk 1 id
        writeInt(output, 16); // subchunk 1 size
        writeShort(output, (short) 1); // audio format (1 = PCM)
        writeShort(output, (short) 1); // number of channels
        writeInt(output, SAMPLE_RATE); // sample rate
        writeInt(output, SAMPLE_RATE * 2); // byte rate
        writeShort(output, (short) 2); // block align
        writeShort(output, (short) 16); // bits per sample
        writeString(output, "data"); // subchunk 2 id
        writeInt(output, rawData.length); // subchunk 2 size

        // Uncomment the following block of code if your PCM file has content in big endian format.
        /*
        // Audio data (conversion big endian -> little endian)
        short[] shorts = new short[rawData.length / 2];
        ByteBuffer.wrap(rawData).order(ByteOrder.LITTLE_ENDIAN).asShortBuffer().get(shorts);
        ByteBuffer bytes = ByteBuffer.allocate(shorts.length * 2);
        for (short s : shorts) {
        	bytes.putShort(s);
        }
        */

        output.write(rawData);
    } finally {
        if (output != null) {
            output.close();
        }
    }

    return outputFile;
}

private void writeInt(final DataOutputStream output, final int value) throws IOException {
    output.write(value >> 0);
    output.write(value >> 8);
    output.write(value >> 16);
    output.write(value >> 24);
}

private void writeShort(final DataOutputStream output, final short value) throws IOException {
    output.write(value >> 0);
    output.write(value >> 8);
}

private void writeString(final DataOutputStream output, final String value) throws IOException {
    for (int i = 0; i < value.length(); i++) {
        output.write(value.charAt(i));
    }
}
```

[audio_record]: https://developer.android.com/reference/android/media/AudioRecord.html
[voice_recorder]: https://github.com/GoogleCloudPlatform/android-docs-samples/blob/master/speech/Speech/app/src/main/java/com/google/cloud/android/speech/VoiceRecorder.java