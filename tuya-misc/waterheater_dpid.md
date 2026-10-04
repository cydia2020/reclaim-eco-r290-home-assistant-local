{\"dpId\":1,\"dpName\":\"ON/OFF\"} 开关 send and report bool (avail: true/false)

{\"dpId\":2,\"dpName\":\"Set Temp\"} 设置温度 send and report int (avail: 10-65 celcius, scale:0 step 1)

{\"dpId\":3,\"dpName\":\"CURRENT TEMPERATURE\"} 当前温度 report only int (avail: -500-1500 scale:0 step 1)

{\"dpId\":4,\"dpName\":\"Working Mode\"} 热水模式，速热模式，假日模式，补救模式 send and report enum (0: ECO mode, 1: Boost mode, 2: Holiday mode, 3: Rescue mode)

{\"dpId\":9,\"dpName\":\"Failure warning\"} report only bitmap max_len=9
avail values: "E04","E06","E05","E02","E07","E01","E08","E03","E11"

{\"dpId\":101,\"dpName\":\"定时专用开关\"} write only bool
{\"dpId\":102,\"dpName\":\"Remaining holiday\"} send and report int tuya value 2 to 199 unit days
{\"dpId\":103,\"dpName\":\"Compressor\"} report only bool
{\"dpId\":104,\"dpName\":\"Fan\"} report only bool



{\"dpId\":105,\"dpName\":\"Heating Element\"} report only bool

{\"dpId\":106,\"dpName\":\"EEV Position\"} report only int 膨胀阀步数 0 to 600

{\"dpId\":107,\"dpName\":\"Tank\"} report only int (temp C) 进水温度 -500 to 2000

{\"dpId\":108,\"dpName\":\"Ambient\"} report only int (temp C) 环境温度 -50 to 100

{\"dpId\":109,\"dpName\":\"Discharge\"} report only int (temp C) 排气温度 -50 to 200

{\"dpId\":110,\"dpName\":\"Coil\"} report only int (temp C) 盘管温度 -50 to 200

{\"dpId\":111,\"dpName\":\"Suction\"} report only int (temp C) 回气温度 -50 to 200 step 1 unit celcius

{\"dpId\":112,\"dpName\":\"Defrost Valve\"} report only bool (binary sensor) 除霜电磁阀

{\"dpId\":113,\"dpName\":\"P39 cycle\"} report only int (xd_cs) 消毒次数 0 to 20000 step 1 unit count

{\"dpId\":114,\"dpName\":\"P39\"} report only int (xd_tm) 消毒倒计时 0 to 9999 step 1 unit minutes

{\"dpId\":115,\"dpName\":\"P40\"} report only int (heat_tm) 热错误倒计时 0 to 9999 step 1 unit minutes

{\"dpId\":116,\"dpName\":\"高温消毒\"} report only bool (disinfect mode binary sensor)
