PWM Breathing LED Experiment
=============================

**The Vivado project for this experiment is "pwm_led".**

This chapter mainly explains how to use PWM to control an LED and achieve a breathing light effect.

Experiment Principle
--------------------

As shown in the figure below, an N-bit counter is used, with a maximum value of 2^N and a minimum value of 0. The counter accumulates with "period" as the step value, and after reaching the maximum value it overflows and enters the next accumulation cycle. When the counter value is greater than "duty", the pulse output is high; otherwise it is low. This produces a pulse output with adjustable duty cycle as shown by the red line in the figure. At the same time, "period" can adjust the pulse frequency and can be understood as the step value of the counter.

.. image:: images/12_media/image1.png

PWM Pulse Width Modulation Diagram

When square waves with different pulse duty cycles are applied to the LED, the LED displays different brightness levels. By continuously adjusting the duty cycle of the square wave, the brightness of the LED can be controlled.

Experiment Design
-----------------

The PWM module design is very simple and has already been explained in the principle section above, so the principle will not be repeated here.

+--------------+--------+----------------------------------------------+
| Signal Name  | Dir    | Description                                  |
+==============+========+==============================================+
| clk          | in     | Clock input                                  |
+--------------+--------+----------------------------------------------+
| rst          | in     | Asynchronous reset input, active high        |
+--------------+--------+----------------------------------------------+
| period       | in     | PWM pulse width period (frequency) control.  |
|              |        | period = PWM output frequency \* (2^N) /     |
|              |        | system clock frequency. The larger N is, the  |
|              |        | higher the frequency precision.              |
+--------------+--------+----------------------------------------------+
| duty         | in     | Duty cycle control, duty cycle = duty /      |
|              |        | (2^N) \* 100%                                |
+--------------+--------+----------------------------------------------+

PWM Module (ax_pwm) Ports

.. code:: verilog

 `timescale 1ns / 1ps
 module ax_pwm
 #(
 parameter N = 16 //pwm bit width 
 )
 (
     input         clk,
     input         rst,
     input[N - 1:0]period,//pwm step value
     input[N - 1:0]duty,//duty value
     output        pwm_out //pwm output
     );
  
 reg[N - 1:0] period_r; - 1:0] duty_r; - 1:0] period_cnt;//period counter
 reg pwm_r;
 assign pwm_out = pwm_r;
 always@(posedge clk or posedge rst)
 begin
     if(rst==1)
     begin
         period_r <= { N {1'b0} };
         duty_r <= { N {1'b0} };
     end
     else
     begin
         period_r <= period;
         duty_r   <= duty;
     end
 end
 //period counter, step is period value
 always@(posedge clk or posedge rst)
 begin
     if(rst==1)
         period_cnt <= { N {1'b0} };
     else
         period_cnt <= period_cnt + period_r;
 end
 
 always@(posedge clk or posedge rst)
 begin
     if(rst==1)
     begin
         pwm_r <= 1'b0;
     end
     else
     begin
         if(period_cnt >= duty_r)//if period counter is bigger or equals to duty value, then set pwm value to high
             pwm_r <= 1'b1;
         else
             pwm_r <= 1'b0;
     end
 end

So how do we achieve the breathing light effect? We know that the breathing light effect is a process of gradually changing from dim to bright, and then from bright to dim. The brightness effect is controlled by the duty cycle, so we mainly need to control the duty cycle, which means controlling the value of duty.

In the test code below, by setting the value of period, the PWM frequency is set to 200Hz. The PWM_PLUS state increases the duty value. If it increases to the maximum value, pwm_flag is set to 1 and the duty value starts decreasing. Once it decreases to the minimum value, the duty value starts increasing again, and this cycle repeats continuously. The PWM_GAP state is the adjustment interval, with a duration of 100us.

.. code:: verilog

 `timescale 1ns / 1ps
 module pwm_test(
                  input clk,//50MHz
                  input rst_n,//low active
              output led//high-off, low-on
                 );
   
 localparam CLK_FREQ = 50 ; T = CLK_FREQ ; ter
 localparam MS_COUNT = CLK_FREQ*1000 ; //1 ms counter
 
 localparam DUTY_STEP  = 32'd100000 ;//duty step
 localparam DUTY_MIN_VALUE = 32'h6fffffff ;//duty minimum value
 localparam DUTY_MAX_VALUE = 32'hffffffff ;//duty maximum value
   
 localparam IDLE    = 0;//IDLE state
 localparam PWM_PLUS  = 1;    //PWM duty plus state
 localparam PWM_MINUS  = 2;    //PWM duty minus state
 localparam PWM_GAP  = 3;    //PWM duty adjustment gap
 
 wire pwm_out;//pwm output
 reg[31:0] period;d minus flag, 0: plus; 1: minus
 
 reg[3:0] state;
 reg[31:0] timer;t counter
 
 assign led = ~pwm_out ; //led low active
 
 always@(posedge clk or negedge rst_n)
 begin
 if(rst_n == 1'b0)
 begin
 period <= 32'd0;
 timer <= 32'd0;
 duty <= 32'd0;
 pwm_flag <= 1'b0 ;
 state <= IDLE;
 end
 else
 case(state)
 IDLE:
 begin
 period <= 32'd17179;   //The pwm step value, pwm 200Hz(period = 200*2^32/50000000)
 state  <= PWM_PLUS;
 duty   <= DUTY_MIN_VALUE;d
 PWM_PLUS :
 begin
 if (duty > DUTY_MAX_VALUE - DUTY_STEP)//if duty is bigger than DUTY MAX VALUE minus DUTY_STEP , begin to minus duty value
 begin
 pwm_flag <= 1'b1 ;
 duty   <= duty - DUTY_STEP ;
 end
 else
 begin
 pwm_flag <= 1'b0 ;
 duty   <= duty + DUTY_STEP ;
 end
 
 state  <= PWM_GAP ;
 end
 PWM_MINUS :
 begin
 if (duty < DUTY_MIN_VALUE + DUTY_STEP)//if duty is little than DUTY MIN VALUE plus duty step, begin to add duty value
 begin
 pwm_flag <= 1'b0 ;
 duty   <= duty + DUTY_STEP ;
 end
 else
 begin
 pwm_flag <= 1'b1 ;
 duty   <= duty - DUTY_STEP ;
 end
 state  <= PWM_GAP ;
 end
 PWM_GAP:
 begin
 if(timer >= US_COUNT*100)      //adjustment gap is 100us
 begin
 if (pwm_flag)
 state <= PWM_MINUS ;
 else
 state <= PWM_PLUS ;
 
 timer <= 32'd0;
 end
 else
 begin
 timer <= timer + 32'd1;
 end
 end
 default:
 begin
 state <= IDLE;ddcase
 end
 
 //Instantiate pwm module
 ax_pwm
 #(
   .N(32)
  ) 
 ax_pwm_m0(
     .clk      (clk),
     .rst      (~rst_n),
     .period   (period),
     .duty     (duty),
     .pwm_out  (pwm_out)
     );
 endmodule

Download and Verification
-------------------------

Generate the bitstream and download the bit file. You can see that PL LED1 produces a breathing light effect. PWM is a commonly used module, for example in fan speed control, motor speed control, and so on.
