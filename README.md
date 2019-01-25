# simple_gcs_file_uploader
a Simple go app to upload write files into google cloud storage buckets. It was originally programmed to diagnose an error caused by a starving cpu so I decided to share it here.


## Compile/Build:
```bash
go build simple_gcs_file_uploader
```

## Run/Usage:
```bash
simple_gcs_file_uploader <bucket_name> <object_name> <file_to_upload_name> <metadata_JSON> <gcs_credentials_file_path>
```

## Code:
```go
// Usage: simple_gcs_file_uploader <bucket_name> <object_name> <file_to_upload_name> <metadata_JSON> <gcs_credentials_file_path>
// simple_gcs_file_uploader.go
// Author: Zack (Abdulrahman) Zaiter
package main

import (
		"encoding/json"
		"io"
		"os"
        "context"
        "fmt"
        "log"
		"cloud.google.com/go/storage"
		"google.golang.org/api/option"
)

func main() {
        ctx := context.Background()
		bucketName := os.Args[1]
		client, err := storage.NewClient(ctx, option.WithCredentialsFile(os.Args[5]))
        if err != nil {
                log.Fatalf("Failed to create client: %v", err)
        }
		bucket := client.Bucket(bucketName)
		inMetadata := []byte(os.Args[4])
		var inMetadataMap map[string]string
		if err := json.Unmarshal(inMetadata, &inMetadataMap);err != nil {
			log.Fatalf("Failed to parse metadata: %v", err)
		}
		obj := bucket.Object(os.Args[2])
		w := obj.NewWriter(ctx)
		w.Metadata = inMetadataMap
		f, err := os.Open(os.Args[3])
		if err != nil {
			log.Fatalf("Failed to read the file: %v", err)
		}
		defer f.Close()
		if _, err := io.Copy(w, f); err != nil {
			log.Fatalf("Failed to write into obj stream: %v", err)
		}
		if err := w.Close(); err != nil {
			log.Fatalf("Failed to close file write stream: %v", err)
		}
        fmt.Printf("Object %v (%v) uploaded to bucket %v with metadata %v.\n", os.Args[2], os.Args[3], bucketName, inMetadataMap)
}
```
