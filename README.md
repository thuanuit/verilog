# verilog
Descript verilog code

module Multiply(y, x, p, en, clk);
  parameter width = 53; // do rong cua input
//  parameter N = width/2 ;// if width is even
  parameter N = (width+1)/2	; // so vong lap voi N-1<= k <= 0 if width is odd, so N = (width+1)/
  input [width-1:0] x; // MD -> Multiplier A
  input [width-1:0] y; // MR -> Multiplier B
  input en;
  input clk;
  output wire [width+width-1:0] p; // output
 
  reg flag;
  wire [width+2:0] y1; // them 1 bit '0' vao y -> y thanh 56 bit
  wire [width-1:0] x; //  
  wire [width-1:0] inv_x; // bu 2 cua x
  reg [2:0] cc[N:0]; // xet Ck 3 bit
  reg [width+1:0] pp[N:0];  // encoding tu Ck -> S = pp
  reg [width+width-1:0] prod; // produces
  reg [width+width-1:0] spp[N:0]; // sign_extend produces 
  
  assign inv_x = {~x[width-1],~x} + 1; // bu 2 cua MD
  assign y1 = {3'b0,y}; // mo rong 56 bit cua Multiplier B
  integer k;
  integer i;
  
	
	always @ (posedge clk)
	begin
	  if (en == 1)
		cc[0] = {y1[1],y1[0],1'b0}; //generate Ck for k=0(special case)
		for(k=1;k<=N;k=k+1)
			cc[k] = {y1[2*k+1],y1[2*k],y1[2*k-1]}; //generate Ck for each k, for k is not 0
 		for(k=0;k<=N;k=k+1)
			begin
				case(cc[k]) //Depending upon Ck, select M,2M,-M,-2M, or 0 as the partial product
					3'b001 , 3'b010 : pp[k] = {2'b00,x};
					3'b011 : pp[k] = {1'b0,x,1'b0};
					3'b100 : pp[k] = {1'b1,inv_x[width-1:0],1'b0};
					3'b101 , 3'b110 : pp[k] = {2'b11,inv_x};
					3'b111 : pp[k] = {2'b11,53'h0};
					3'b000 : pp[k] = {2'b00,53'h0};
					default : pp[k] = 0;
				endcase
				spp[k] = $signed(pp[k]);//sign extend
					for(i=0;i<k;i=i+1)
						spp[k] = {spp[k],2'b00}; //multiply by 2 to the power x or shifting operation
			end //for(kk=0;kk<N;kk=kk+1)
		prod = spp[0];
		for(k=1;k<N;k=k+1)
			prod = prod + spp[k]; //add partial products to get result
	end
		assign p = prod;
    
endmodule 
