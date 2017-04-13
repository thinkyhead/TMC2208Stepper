# Table of contents
[Headers](#headers)
[Emphasis](#emphasis)

# TMC2208Stepper
Arduino library for Trinamic TMC2208 Stepper driver


### Example
```cpp
#define EN_PIN    13 								// LOW: Driver enabled. HIGH: Driver disabled
#define STEP_PIN  54 								// Step on rising edge

#include <TMC2208Stepper.h>							// Include library
TMC2208Stepper driver = TMC2208Stepper(&Serial);	// Create driver and use
													// HardwareSerial0 for communication

void setup() {
	Serial.begin(115200);							// Init used serial port
	while(!Serial);									// Wait for port to be ready

	// Prepare pins
	pinMode(EN_PIN, OUTPUT);
	pinMode(STEP_PIN, OUTPUT);

	driver.pdn_disable(1);							// Use PDN/UART pin for communication
	driver.I_scale_analog(0);						// Adjust current from the registers
	driver.rms_current(500);						// Set driver current 500mA
	driver.toff(0x2);								// Enable driver

	digitalWrite(13, LOW);							// Enable driver
}

void loop() {
	digitalWrite(STEP_PIN, !digitalRead(STEP_PIN)); // Step
	delay(10);
}
```

## Helper functions

Function | Description
------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
void rms_current(<br>**uint16_t<br>float<br>float**<br>) 	| Set motor RMS current<br>Arguments:<br>**uint16_t** current_rms<br><i>Optional:</i><br>**float** hold current multiplier (default=0.5)<br>**float** sense resistor value (default=0.11)
void microsteps(uint16_t)									| [0..256] Set number of microsteps

## RW: GCONF
Function | Description
----------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
bool GCONF(uint32_t *)		| Read GCONF register
void GCONF(uint32_t)  		| Write to the GCONF register
void I_scale_analog(bool)	| (Reset default=1)<br>											0: Use internal reference derived from 5VOUT<br>											1: Use voltage supplied to VREF as current reference
void internal_Rsense(bool)	| (Reset default: OTP)<br>										0: Operation with external sense resistors<br>												1: Internal sense resistors. Use current supplied into VREF as reference<br>	for internal sense resistor. VREF pin internally is driven to GND in this mode.
void en_spreadCycle(bool)	| (Reset default: OTP)<br>										0: stealthChop PWM mode enabled (depending on velocity thresholds).<br>						Initially switch from off to on state while in stand still, only.<br>			1: spreadCycle mode enabled
void shaft(bool)			| 1: Inverse motor direction
void index_otpw(bool)		| 0: INDEX shows the first microstep position of sequencer<br>	1: INDEX pin outputs overtemperature prewarning flag (otpw) instead
void index_step(bool)		| 0: INDEX output as selected by index_otpw<br>					1: INDEX output shows step pulses from internal pulse generator (toggle upon each step)
void pdn_disable(bool)		| 0: PDN_UART controls standstill current reduction<br>			1: PDN_UART input function disabled. Set this bit, when using the UART interface!
void mstep_reg_select(bool)	| 0: Microstep resolution selected by pins MS1, MS2<br>			1: Microstep resolution selected by MSTEP register
void multistep_filt(bool)	| (Reset default=1)<br>											0: No filtering of STEP pulses<br>															1: Software pulse generator optimization enabled when fullstep<br>				frequency > 750Hz (roughly). TSTEP shows filtered step time values when active.

## R+WC: GSTAT
Function | Description
----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
void GSTAT(uint32_t)		| Write to GSTAT register
bool GSTAT(uint32_t*)		| Read GSTAT register
void reset(bool)			| 1: Indicates that the IC has been reset since the last<br>	read access to GSTAT. All registers have been cleared to reset values.
void drv_err(bool)			| 1: Indicates, that the driver has been shut down due to<br>	overtemperature or short circuit detection since the last read access. Read DRV_STATUS<br>	for details. The flag can only be cleared when all error conditions are cleared.
void uv_cp(bool)			| 1: Indicates an undervoltage on the charge pump. The<br>		driver is disabled in this case. This flag is not latched and thus does not need to be cleared.

## R: IFCNT
Function | Description
----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
bool IFCNT(uint32_t*)		| Interface transmission counter. This register becomes<br>		incremented with each successful UART interface write access. Read out to check the serial<br>	transmission for lost data. Read accesses do not change the content.<br>		The counter wraps around from 255 to 0.

## W: SLAVECONF
Function | Description
----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
void SLAVECONF(uint32_t)	| Write register
bool SLAVECONF(uint32_t*)	| Read register
void senddelay(uint8_t)		| SENDDELAY for read access (time until reply is sent):<br>		0, 1: 8 bit times<br>																		2, 3: 3*8 bit times<br>															4, 5: 5*8 bit times<br>			6, 7: 7*8 bit times<br>			8, 9: 9*8 bit times<br>			10, 11: 11*8 bit times<br>			12, 13: 13*8 bit times<br>			14, 15: 15*8 bit times

## W: OTP_PROG
Function | Description
----------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
void OTP_PROG(uint32_t)		| OTP_PROGRAM – OTP programming Write access programs OTP<br>	memory (one bit at a time), Read access refreshes read data from OTP after a write<br>		2..0 OTPBIT<br>																	Selection of OTP bit to be programmed to the selected byte location (n=0..7: programs bit n to a logic 1)<br>5..4 OTPBYTE<br>Selection of OTP programming location (0, 1 or 2)<br>15..8 OTPMAGIC<br>Set to 0xbd to enable programming. A programming time of minimum 10ms per bit is recommended (check by reading OTP_READ).

## R: OTP_READ
Function | Description
-|-

## R: IOIN
Function | Description
-|-
bool IOIN(uint32_t*)			| INPUT (Reads the state of all input pins available)
bool enn()						| ENN
bool ms1()						| MS1
bool ms2()						| MS2
bool diag()						| DIAG
bool pdn_uart()					| PDN_UART
bool step()						| STEP
bool sel_a()					| SEL_A: Driver type<br>									1: TMC220x<br>																				0: TMC222x
bool dir()						| DIR
uint8_t version()				| VERSION: 0x20=first version of the IC<br>					Identical numbers mean full digital compatibility.

## RW: FACTORY_CONF
Function | Description
--------------------------------|---------------------------------------
void FACTORY_CONF(uint32_t)		| Write register
bool FACTORY_CONF(uint32_t*)	| Read register
void fclktrim(uint8_t)			| 0…31: Lowest to highest clock frequency. Check at<br>		charge pump output. The frequency span is not guaranteed, but it is tested, that tuning<br>	to 12MHz internal clock is possible. The devices come preset to 12MHz clock<br>	frequency by OTP programming.
void ottrim(uint8_t)			| %00: OT=143°C, OTPW=120°C<br>								%01: OT=150°C, OTPW=120°C<br>																%10: OT=150°C, OTPW=143°C<br>													%11: OT=157°C, OTPW=143°C

## W: IHOLD_IRUN
Function | Description
--------------------------------|---------------------------------------
void IHOLD_IRUN(uint32_t)		| 
bool IHOLD_IRUN(uint32_t*)		| 
void ihold(uint8_t)				| (Reset default: OTP)<br>Standstill current (0=1/32 … 31=32/32)<br>In combination with stealthChop mode, setting IHOLD=0 allows to choose freewheeling or coil short circuit (passive braking) for motor stand still.
void irun(uint8_t)				| (Reset default=31)<br>Motor run current (0=1/32 … 31=32/32)<br>Hint: Choose sense resistors in a way, that normal IRUN is 16 to 31 for best microstep performance.
void iholddelay(uint8_t)		| (Reset default: OTP)<br>Controls the number of clock cycles for motor power down after standstill is detected (stst=1) and TPOWERDOWN has expired. The smooth transition avoids a motor jerk upon power down.<br>0: instant power down<br>1..15: Delay per current reduction step in multiple of 2^18 clocks

## W: TPOWERDOWN
Function | Description
--------------------------------|---------------------------------------
void TPOWERDOWN(uint32_t)		| (Reset default=20)<br>									Sets the delay time from stand still (stst) detection to motor current power down.<br>		Time range is about 0 to 5.6 seconds.<br>0…((2^8)-1) * 2^18 tCLK<br>Attention: A minimum setting of 2 is required to allow automatic tuning of stealthChop PWM_OFFS_AUTO.
bool TPOWERDOWN(uint32_t*) 		|

## R: TSTEP
Function | Description
--------------------------------|---------------------------------------
bool TSTEP(uint32_t*)			| Actual measured time between two 1/256 microsteps derived from the step input frequency in units of 1/fCLK. Measured value is (2^20)-1 in case of overflow or stand still.<br>														The TSTEP related threshold uses a hysteresis of 1/16 of the compare value to compensate for jitter in the clock or the step frequency: (Txxx*15/16)-1 is the lower compare value for each TSTEP based comparison.<br>This means, that the lower switching velocity equals the calculated setting, but the upper switching velocity is higher as defined by the hysteresis setting.

## W: TPWMTHRS
Function | Description
--------------------------------|---------------------------------------
void TPWMTHRS(uint32_t)			| Sets the upper velocity for stealthChop voltage PWM mode.<br>TSTEP ≥ TPWMTHRS<br>- stealthChop PWM mode is enabled, if configured<br>When the velocity exceeds the limit set by TPWMTHRS, the driver switches to spreadCycle.<p>0: Disabled
bool TPWMTHRS(uint32_t*)

## W: VACTUAL
Function | Description
--------------------------------|---------------------------------------
void VACTUAL(uint32_t)			| VACTUAL allows moving the motor by UART control.<br>It gives the motor velocity in +-(2^23)-1 [μsteps / t]<br>0: Normal operation. Driver reacts to STEP input.<br>/=0: Motor moves with the velocity given by VACTUAL. Step pulses can be monitored via INDEX output. The motor direction is controlled by the sign of VACTUAL.
bool VACTUAL(uint32_t *)

## R: MSCNT
Function | Description
--------------------------------|---------------------------------------
bool MSCNT(uint32_t*)			| Microstep counter. Indicates actual position in the microstep table for CUR_A. CUR_B uses an offset of 256 into the table. Reading out MSCNT allows determination of the motor position within the electrical wave.

## R: MSCURACT
Function | Description
--------------------------------|---------------------------------------
bool MSCURACT(uint32_t*)
uint16_t cur_a()				| CUR_A (signed):<br>Actual microstep current for motor phase A as read from the internal sine wave table (not scaled by current setting)
uint16_t cur_b()				| CUR_B (signed):<br>Actual microstep current for motor phase B as read from the internal sine wave table (not scaled by current setting)

## RW: CHOPCONF
Function | Description
--------------------------------|---------------------------------------
void CHOPCONF(uint32_t)			|
void toff(uint8_t)				| off time and driver enable<p>Off time setting controls duration of slow decay phase<br>NCLK= 12 + 32*TOFF<br>%0000: Driver disable, all bridges off<br>%0001: 1 – use only with TBL ≥ 2<br>%0010 … %1111: 2 … 15<br>(Default: OTP, resp. 3 in stealthChop mode)
void hstrt(uint8_t)				| hysteresis start value added to HEND<p>%000 … %111:<br>Add 1, 2, …, 8 to hysteresis low value HEND<br>(1/512 of this setting adds to current setting)<br>Attention: Effective HEND+HSTRT ≤ 16.<br>Hint: Hysteresis decrement is done each 16 clocks<br>(Default: OTP, resp. 0 in stealthChop mode)
void hend(uint8_t)				| hysteresis low value<p>%0000 … %1111:<br>Hysteresis is -3, -2, -1, 0, 1, …, 12<br>(1/512 of this setting adds to current setting)<br>This is the hysteresis value which becomes used for the hysteresis chopper.<br>(Default: OTP, resp. 5 in stealthChop mode)
void tbl(uint8_t)				| blank time select<p>%00 … %11:<br>Set comparator blank time to 16, 24, 32 or 40 clocks<br>Hint: %00 or %01 is recommended for most applications<br>(Default: OTP)
void vsense(bool)				| sense resistor voltage based current scaling<p>0: Low sensitivity, high sense resistor voltage<br>1: High sensitivity, low sense resistor voltage
void mres(uint8_t)				| micro step resolution<p>%0000:<br>Native 256 microstep setting.<br>%0001 … %1000:<br>128, 64, 32, 16, 8, 4, 2, FULLSTEP<br>Reduced microstep resolution.<br>The resolution gives the number of microstep entries per sine quarter wave.<br>When choosing a lower microstep resolution, the driver automatically uses microstep positions which result in a symmetrical wave.<br>Number of microsteps per step pulse = 2^MRES<br>(Selection by pins unless disabled by GCONF. mstep_reg_select)
void intpol(bool)				| interpolation to 256 microsteps<p>1: The actual microstep resolution (MRES) becomes extrapolated to 256 microsteps for smoothest motor operation.<br>(Default: 1)
void dedge(bool)				| enable double edge step pulses<p>1: Enable step impulse at each step edge to reduce step frequency requirement. This mode is not compatible with the step filtering function (multistep_filt)
void diss2g(bool)				| 0: Short to GND protection is on<br>1: Short to GND protection is disabled
void diss2vs(bool)				| 0: Short protection low side is on<br>1: Short protection low side is disabled
bool CHOPCONF(uint32_t*)		| 
uint8_t toff()					| 
uint8_t hstrt()					| 
uint8_t hend()					| 
uint8_t tbl()					| 
bool vsense()					| 
uint8_t mres()					| 
bool intpol()					| 
bool dedge()					| 
bool diss2g()					| 
bool diss2vs()					| 

## R: DRV_STATUS
Function | Description
--------------------------------|---------------------------------------
bool DRV_STATUS(uint32_t*)		| 
bool otpw()						| 1: The selected overtemperature pre-warning threshold is exceeded.
The overtemperature pre-warning flag is common for both bridges.
bool ot()						| 1: The selected overtemperature limit has been reached. Drivers become disabled until otpw is also cleared due to cooling down of the IC.
The overtemperature flag is common for both bridges.
bool s2ga()						| 1: Short to GND detected on phase A or B. The driver becomes disabled. The flags stay active, until the driver is disabled by software (TOFF=0) or by the ENN input. Flags are separate for both chopper modes.
bool s2gb()						| 
bool s2vsa()					| 1: Short on low-side MOSFET detected on phase A or B. The driver becomes disabled. The flags stay active, until the driver is disabled by software (TOFF=0) or by the ENN input. Flags are separate for both chopper modes.
bool s2vsb()					| 
bool ola()						| 1: Open load detected on phase A or B.
Hint: This is just an informative flag. The driver takes no action upon it. False detection may occur in fast motion and standstill. Check during slow motion, only.
bool olb()						| 
bool t120()						| 1: Temperature threshold is exceeded
bool t143()						| 
bool t150()						| 
bool t157()						| 
bool stealth()					| 1: Driver operates in stealthChop mode<br>0: Driver operates in spreadCycle mode
bool stst()						| This flag indicates motor stand still in each operation mode. This occurs 2^20 clocks after the last step pulse.

## RW: PWMCONF
Function | Description
--------------------------------|---------------------------------------
void PWMCONF(uint32_t)			| 
void pwm_ofs(uint8_t)			| User defined amplitude (offset)<p>User defined PWM amplitude offset (0-255) related to full motor current (CS_ACTUAL=31) in stand still.<br>(Reset default=36)<p>When using automatic scaling (pwm_autoscale=1) the value is used for initialization, only. The autoscale function starts with PWM_SCALE_AUTO=PWM_OFS and finds the required offset to yield the target current automatically.<p>PWM_OFS = 0 will disable scaling down motor current below a motor specific lower measurement threshold. This setting should only be used under certain conditions, i.e. when the power supply voltage can vary up and down by a factor of two or more. It prevents the motor going out of regulation, but it also prevents power down below the regulation limit.<p>PWM_OFS > 0 allows automatic scaling to low PWM duty cycles even below the lower regulation threshold. This allows low (standstill) current settings based on the actual (hold) current scale (register IHOLD_IRUN).
void pwm_grad(uint8_t)			| User defined amplitude gradient<p>Velocity dependent gradient for PWM amplitude:<br>PWM_GRAD * 256 / TSTEP<br>This value is added to PWM_AMPL to compensate for the velocity-dependent motor back-EMF.<p>With automatic scaling (pwm_autoscale=1) the value is used for first initialization, only. Set PWM_GRAD to the application specific value (it can be read out from PWM_GRAD_AUTO) to speed up the automatic tuning process. An approximate value can be stored to OTP by programming OTP_PWM_GRAD.
void pwm_freq(uint8_t)			| PWM frequency frequency<p>%00: fPWM=2/1024 fCLK<br>%01: fPWM=2/683 fCLK<br>%10: fPWM=2/512 fCLK<br>%11: fPWM=2/410 fCLK
void pwm_autoscale(bool)		| PWM automatic amplitude scaling<p>0: User defined feed forward PWM amplitude. The current settings IRUN and IHOLD have no influence!<br>The resulting PWM amplitude (limited to 0…255) is:<br>PWM_OFS * ((CS_ACTUAL+1) / 32) + PWM_GRAD * 256 / TSTEP<br>1: Enable automatic current control (Reset default)
void pwm_autograd(bool)			| PWM automatic gradient adaptation<p>0: Fixed value for PWM_GRAD<br>(PWM_GRAD_AUTO = PWM_GRAD)<br>1: Automatic tuning (only with pwm_autoscale=1)<br>PWM_GRAD_AUTO is initialized with PWM_GRAD and becomes optimized automatically during motion.<br>Preconditions<br>1. PWM_OFS_AUTO has been automatically initialized. This requires standstill at IRUN for >130ms in order to a) detect standstill b) wait > 128 chopper cycles at IRUN and c) regulate PWM_OFS_AUTO so that -1 < PWM_SCALE_AUTO < 1<br>2. Motor running and 1.5 * PWM_OFS_AUTO < PWM_SCALE_SUM < 4* PWM_OFS_AUTO and PWM_SCALE_SUM < 255.<br>Time required for tuning PWM_GRAD
void freewheel(uint8_t)			| Allows different standstill modes<p>Stand still option when motor current setting is zero (I_HOLD=0).<br>%00: Normal operation<br>%01: Freewheeling<br>%10: Coil shorted using LS drivers<br>%11: Coil shorted using HS drivers
void pwm_reg(uint8_t)			| Regulation loop gradient<p>User defined maximum PWM amplitude change per half wave when using pwm_autoscale=1. (1…15):<br>1: 0.5 increments (slowest regulation)<br>2: 1 increment (default with OTP2.1=1)<br>3: 1.5 increments<br>4: 2 increments<br>…<br>8: 4 increments (default with OTP2.1=0)<br>...<br>15: 7.5 increments (fastest regulation)
void pwm_lim(uint8_t)			| PWM automatic scale amplitude limit when switching on<p>Limit for PWM_SCALE_AUTO when switching back from spreadCycle to stealthChop. This value defines the upper limit for bits 7 to 4 of the automatic current control when switching back. It can be set to reduce the current jerk during mode change back to stealthChop.<br>It does not limit PWM_GRAD or PWM_GRAD_AUTO offset.<br>(Default = 12)
bool PWMCONF(uint32_t*)			| 
uint8_t pwm_ofs()				| 
uint8_t pwm_grad()				| 
uint8_t pwm_freq()				| 
bool pwm_autoscale()			| 
bool pwm_autograd()				| 
uint8_t freewheel()				| 
uint8_t pwm_reg()				| 
uint8_t pwm_lim()				| 

## R: PWM_SCALE
Function | Description
--------------------------------|---------------------------------------
bool PWM_SCALE(uint32_t*)		| 
uint8_t pwm_scale_sum()			| Results of stealthChop amplitude regulator. These values can be used to monitor automatic PWM amplitude scaling (255=max. voltage).<p>bit 7… 0 PWM_SCALE_SUM:<br>Actual PWM duty cycle. This value is used for scaling the values CUR_A and CUR_B read from the sine wave table.<p>bit 24… 16 PWM_SCALE_AUTO:<br>9 Bit signed offset added to the calculated PWM duty cycle. This is the result of the automatic amplitude regulation based on current measurement.
int16_t pwm_scale_auto()		| These automatically generated values can be read out in order to determine a default / power up setting for PWM_GRAD and PWM_OFS.<p>bit 7… 0 PWM_OFS_AUTO:<br>Automatically determined offset value<p>bit 23… 16 PWM_GRAD_AUTO:<br>Automatically determined gradient value
