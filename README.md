The following code computes row sums of matrix data[100][100] by 2 processes.
--------------------------
#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>
#define generate_data(i,j) (i)+(j)*(j)
int main( int argc, char **argv)
{
 int i, j, pid, np, mtag, count;
 double t0, t1 ;
 int data[100][100], row_sum[100] ;
 MPI_Status status;
 MPI_Request req_s, req_r;
 MPI_Init( &argc, &argv );
 MPI_Comm_rank(MPI_COMM_WORLD, &pid);
 MPI_Comm_size(MPI_COMM_WORLD, &np);
 if(pid == 0) { // generate data[]
 for(i=0; i<50; i++)
 for(j=0; j<100; j++)
 data[i][j] = generate_data(i,j) ;
 mtag = 1 ;
 MPI_Isend(data, 5000, MPI_INT, 1, mtag, MPI_COMM_WORLD, &req_s) ;
 for(i=50; i<100; i++)
 for(j=0; j<100; j++)
 data[i][j] = generate_data(i,j) ;
 for(i=50; i<100; i++) {
 row_sum[i] = 0 ;
 for(j=0; j<100; j++)
 row_sum[i] += data[i][j] ;
 }
 MPI_Wait(&req_s, &status) ;
 /*** receive computed row_sums from pid 1 ***/
 mtag = 2 ;
 MPI_Recv(row_sum, 50, MPI_INT, 1, mtag, MPI_COMM_WORLD, &status) ;
 for(i=0; i<100; i++) {
 printf(" %d ", row_sum[i]) ;
 if(i%10 ==9) printf("\n");
 }
 }
 else { /*** pid == 1 ***/
 mtag = 1 ;
 MPI_Recv(data, 5000, MPI_INT, 0, mtag, MPI_COMM_WORLD, &status) ;
 for(i=0; i<50; i++) {
 row_sum[i] = 0 ;
 for(j=0; j<100; j++)
 row_sum[i] += data[i][j] ;
 }
 /*** Send computed row_sums to pid 0 ***/
 mtag = 2 ;
 MPI_Send(row_sum, 50, MPI_INT, 0, mtag, MPI_COMM_WORLD) ;
 }
 MPI_Finalize();
 return 1;
} 

/****************** End of function main() ********************/
For this assignment, please rewrite the code so that on pid==1, there is significant overlapping of receiving some rows with the computation of doing rowsummation of some other rows. 
Since the requirement is that there is significant overlapping of communication with computation on pid==1, so do not re-assign the sub-computations between the two processes, 
i.e. the process with pid==1 does only the row sums for row 0 to row 49, where the data needed by tis process (i.e. pid==1) for the row sums are to be received from the process with pid==0. 
This assignment asks you to achieve overlapping of receiving of some rows (from row 0 to row 49) with the computation row sums for some other rows (from row 0 to row 49).
