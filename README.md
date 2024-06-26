# TODO DESCRIPTION

## Converting Zoom Audio to Speech-to-Text Compatible audio files

1. Do a Zoom meeting and record
2. Convert audio file (.mp4a) to wav:  `$ ffmpeg -i whatever.mp4a whatever.wav`
3. Then manually ran it through transcription (speech-to-text)
4. Stored output in /digest/transcription/api-response/zoom.json


GCP tool: https://console.cloud.google.com/speech/transcriptions/list?project=digest-427612

Sample API request:
```
import (
  "context"
  "log"

  speech "cloud.google.com/go/speech/apiv2"
  speechpb "cloud.google.com/go/speech/apiv2/speechpb"
)

func main() {
  ctx := context.Background()
  c, err := speech.NewClient(ctx)
  if err != nil {
    log.Fatalf("failed to create client: %v", err)
  }
  defer c.Close()

  // The path to the remote audio file to transcribe
  fileUri := "gs://digest-bucket/audio-files/zoom.wav"
  // The path to the transcript result folder.
  outputUri := "gs://digest-bucket/transcripts"
  // Recognizer resource name.
  recognizer := "projects/digest-427612/locations/global/recognizers/_"

  config := &speechpb.RecognitionConfig{
    DecodingConfig: &speechpb.RecognitionConfig_ExplicitDecodingConfig{
      ExplicitDecodingConfig: &speechpb.ExplicitDecodingConfig{
          Encoding: speechpb.ExplicitDecodingConfig_LINEAR16,
          SampleRateHertz: 32000,
          AudioChannelCount: 1,
      },
    },
    Model: "long",
    LanguageCodes: []string{"en-US"},
    Features: &speechpb.RecognitionFeatures {
      EnableWordTimeOffsets: true,
      EnableWordConfidence: true,
    },
  }

  audioFiles := []*speechpb.BatchRecognizeFileMetadata{
    &speechpb.BatchRecognizeFileMetadata{
      AudioSource: &speechpb.BatchRecognizeFileMetadata_Uri{
        Uri: fileUri,
      },
    },
  }
  outputConfig := &speechpb.RecognitionOutputConfig{
    Output: &speechpb.RecognitionOutputConfig_GcsOutputConfig{
      GcsOutputConfig: &speechpb.GcsOutputConfig{
        Uri: outputUri,
      },
    },
  }
  req := &speechpb.BatchRecognizeRequest{
    Recognizer:              recognizer,
    Config:                  config,
    Files:                   audioFiles,
    RecognitionOutputConfig: outputConfig,
  }
  _, err = c.BatchRecognize(ctx, req)
  if err != nil {
    log.Fatalf("failed to create BatchRecognize: %v", err)
  }
}
```
