# Encryptor Decrypt flag 
```go

package main

import (
    "bufio"
    "fmt"
    "log"
    "math/big"
    "os"
)


func main() {
    f, err := os.Open("flag.bin")
    if err != nil {
        fmt.Println("Failed", err)
    }
    defer f.Close()
    stat, stat_err := f.Stat()
    if stat_err != nil {
        fmt.Println(stat_err)
    }
    size := stat.Size()
    b := make([]byte, size+1)
    bufio.NewReader(f).Read(b)

    flag := big.NewInt(0)
    for i := 0; i < int(size); i++ {
        for j := 0; j <= 0x3f; j++ {
            for m := 0; m <= 1; m++ {
                if ((2 * j) + m) == int(b[i]) {
                    flag.Lsh(flag, 6)
                    flag.Add(flag, big.NewInt(int64(j)))
                    m = 2
                    j = 0x3f + 1
                }
            }
        }
    }
    f2, _ := os.OpenFile(
        "flag.jpg",
        os.O_WRONLY|os.O_TRUNC|os.O_CREATE,
        0666,
    )
    defer f2.Close()
    bytesWritten, err := f2.Write(flag.Bytes())
    if err != nil {
        log.Fatal(err)
    }
    log.Printf("Wrote %d bytes.\n", bytesWritten)
    
}
```
