# Goback-N




### Pseudocode: [gobabk-n.c](TRASH/goback-n.c)

**Question:**
```
Station A needs to send a message consisting of 6 packets 
to station B  using a sliding window (window size 5) and 
Go Back N error control strategy. All packets are ready 
and immediately available for transmission.

If every 3rd packet that A transmits gets lost 
(but no ACKs from B ever get lost), then what is 
the number of packets that A will transmit for 
sending the message to B?
```
**Sol:**

    frame_size = 6
    N = 5 (window size)
    n = 3


|S.No|Frame                      |  Ack(Rn-1) |
|---:|:--------------------------|-----------:|
|1.  | frame 1 sent              |  1         |
|2.  | frame 2 sent              |  2         |
|3.  | frame 3 sent (courrupted) |  2         |
|4.  | frame 4 sent              |  2         |
|5.  | frame 5 sent              |  2         |
|6.  | frame 6 sent (courrupted) |  2         |
|7.  | frame 3 sent `(timeout)`  |  3         |
|8.  | frame 4 sent              |  4         |
|9.  | frame 5 sent (courrupted) |  4         |
|10. | frame 6 sent              |  4         |
|11. | frame 5 sent `(timeout)`  |  5         |
|12. | frame 6 sent (courrupted) |  5         |
|13. | frame 6 sent `(timeout)`  |  6         |


### program:

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
/*PSEUDOCODE from (https://en.wikipedia.org/wiki/Go-Back-N_ARQ)

function receiver is
    Rn := 0
    Do the following forever:
        if the packet received = Rn and the packet is error free then
            Accept the packet and send it to a higher layer
            Rn := Rn + 1
        else
            Refuse packet
        Send a Request for Rn : ack = Rn - 1 i.e. ack+1


function sender is
    Sb := 0
    Sm := N + 1
    Repeat the following steps forever:
        if you receive a request number where Rn > Sb i.e. (ack >= Sb) then
            Sm := (Sm − Sb) + Rn
            Sb := Rn
        if no packet is in transmission then
            Transmit a packet where Sb ≤ Sn ≤ Sm.  
            Packets are transmitted in order.
*/

// Global variables
int frame_size;
int N  = 10;      // Window size
int Rn = 0;       // Request number `Rn = Ack + 1`
int Sn = 0;       // `[Sb:Sm)` Sequence number  [0 1 2 3 4 5]
int Sb = 0;       // `Sb = 0`Sequence base      (0)1 2 3 4 5
int Sm = 11;      // `Sm = N + 1` Sequence max   0 1 2 3 4 5 (6)

int main() {
    int count = 0,n;
    printf("Enter size of frame: ");
    scanf("%d",&frame_size); //6
    printf("Enter sliding window size: ");
    scanf("%d",&N); //5
    Sm = N; //0 --> (N-1) + 1
    printf("Enter which nth frame is courrupted: ");
    scanf("%d",&n); //3

    if(N <= 0 || frame_size < N || n < 1) {
            printf("Invalied inputs.");
            return 1;
    } printf("GoBack-%d ARQ:\n",N);

    for (int ack = -1; ack < frame_size - 1; printf(" //timeout")) 
    {
        for ( Sn = Sb; Sn < Sm ; Sn++) {
            count++; //frame sent
            ack = (count%n == 0)? ack : (Sn == Sb)? Sn : ack;

            printf("\n%2d. Frame %2d is send. Ack:%2d",count,Sn,ack);
                // ack = (count%n == 0)? ack,printf(" (courrupted)") : (Sn == Sb)? Sn,printf(" //timeout") : ack;
                // printf(" [%2d(%2d)%2d:%2d]",Sb,Sn,Sm,ack);
            Sm = (ack + (Sm - Sb + 1) >= frame_size)? frame_size : ack + (Sm - Sb + 1);
            Sb = ack + 1;
                // printf(" --> [%2d(%2d)%2d:%2d]",Sb,Sn+1,Sm,ack);
            if (count%n == 0){
                printf(" (courrupted!)");
            }
        }
    }
    return 0;
}
```
### Output
```
..\CN-Lab\Lab-6> cc GoBack-N.c -std=c99
..\CN-Lab\Lab-6> ./a.out

Enter size of frame: 6
Enter sliding window size: 5
Enter which nth frame is courrupted: 3

GoBack-5 ARQ:
 1. Frame  1 is send. Ack: 1
 2. Frame  2 is send. Ack: 2
 3. Frame  3 is send. Ack: 2 (courrupted!)
 4. Frame  4 is send. Ack: 2
 5. Frame  5 is send. Ack: 2
 6. Frame  6 is send. Ack: 2 (courrupted!) //timeout
 7. Frame  3 is send. Ack: 3
 8. Frame  4 is send. Ack: 4
 9. Frame  5 is send. Ack: 4 (courrupted!)
10. Frame  6 is send. Ack: 4 //timeout
11. Frame  5 is send. Ack: 5
12. Frame  6 is send. Ack: 5 (courrupted!) //timeout
13. Frame  6 is send. Ack: 6 //timeout
..CN-Lab\Lab-6>
```

## Another Aproch

Thanks to [@chakradharlucky](https://github.com/chakradharlucky) for supprting to the program 🤝.

[`</code>:`513GobackN.c](Lab-6/513GobackN.c)

### program:
```c
#include<stdio.h>
void main(){
    int frameSize, windowSize, n, seqMax, seqNext, count = 0, flg = 0, ack = 0;
    
    printf("Enter size of frame: ");
    scanf("%d",&frameSize); 
    printf("Enter sliding window size: ");
    scanf("%d",&windowSize);
    printf("Enter which nth frame is courrupted: ");
    scanf("%d",&n);
    printf("GoBack-%d ARQ:\n",windowSize);

    if(windowSize <= 0 || frameSize < windowSize || n < 1) {
        printf("Invalied inputs.");
        return;
    }
    while(ack<frameSize) {
        seqMax = frameSize;
        flg = 0;
        for( seqNext = ack + 1 ; (seqNext <= seqMax) && (seqNext <= frameSize) ; seqNext++) {
            count++;
            if((count%n==0) && (flg == 0)) {
                flg = 1;
                seqMax = seqNext + windowSize - 1;
            }
            if(flg == 0)
                ack++;
            
            printf("\n%2d. Frame %2d is send. Ack:%2d",count,seqNext,ack);
            if(count%n==0)
                printf(" (courrupted!)");
        }
    }
}
```

<div style="page-break-after: always; visibility: hidden"></div>
<!-- <div style="page-break-after: always; visibility: hidden">\pagebreak</div> -->

### Output


```
..\CN-Lab\Lab-6> cc 513GoBack-N.c -std=c99
..\CN-Lab\Lab-6> ./a.out

Enter size of frame: 10
Enter sliding window size: 3
Enter which nth frame is courrupted: 5
GoBack-3 ARQ:

 1. Frame  1 is send. Ack: 1
 2. Frame  2 is send. Ack: 2
 3. Frame  3 is send. Ack: 3
 4. Frame  4 is send. Ack: 4
 5. Frame  5 is send. Ack: 4 (courrupted!)
 6. Frame  6 is send. Ack: 4
 7. Frame  7 is send. Ack: 4
 8. Frame  5 is send. Ack: 5
 9. Frame  6 is send. Ack: 6
10. Frame  7 is send. Ack: 6 (courrupted!)
11. Frame  8 is send. Ack: 6
12. Frame  9 is send. Ack: 6
13. Frame  7 is send. Ack: 7
14. Frame  8 is send. Ack: 8
15. Frame  9 is send. Ack: 8 (courrupted!)
16. Frame 10 is send. Ack: 8
17. Frame  9 is send. Ack: 9
18. Frame 10 is send. Ack:10
..\CN-Lab\Lab-6> ./a.out
```
