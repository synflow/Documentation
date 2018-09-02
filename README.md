# Introducing Cx

Cx aims to bring digital hardware design within reach of any developer. The language focuses on being intuitive, easy to learn and use, and elegant; we try to honor the [principle of least surprise](http://en.wikipedia.org/wiki/Principle_of_least_surprise).

An application written in Cx is described as a set of sequential tasks connected together and executed concurrently, which is probably the most natural way to describe parallelism that corresponds to how hardware behaves. The language has a familiar C-like syntax (but is not based on C), features [strong bit-accurate static typing](/documentation/concepts#type-system), and allows cycle-accurate behavior to be [implicitly expressed with structured code](/documentation/concepts#execution).

# Why should I learn Cx?

If you want to get started with hardware design and FPGAs, Cx is the language that is the most intuitive and the easiest to learn and use. If you already know hardware design and are looking for better solutions than VHDL and Verilog, Cx will allow you to get more work done with much less pain.
    
# A modern language

Cx is a clean slate design of a programming language rather than being hacked on top of an existing language inheriting decades of history (C), or being a subset of a language that is ill-suited to hardware (pretty much any software language). It has been designed exclusively for creating digital hardware systems and applications in accordance with the needs of software and hardware engineers.

    task SobelKernel {
      import support.BufferConstants.*;
    
      in pix_t p00, p01, p02,
      p10, p11, p12,
      p20, p21, p22;
    
      out pix_t res;
    
      void loop() {
        pix_t
        p00 = p00.read(), p01 = p01.read(), p02 = p02.read(),
        p10 = p10.read(), /*    p11.read */ p12 = p12.read(),
        p20 = p20.read(), p21 = p21.read(), p22 = p22.read();
    
        i15 Gx =
          -p00     + p02
          -2 * p10 + 2 * p12
          -p20     + p22;
        i15 Gy =
          p00 + 2 * p01 + p02
          -p20 - 2 * p21 - p22;
    
        res.write((pix_t) (abs(Gx) + abs(Gy)));
      }
    
      const int abs(int x) {
        return (int) (x < 0 ? -x : x);
      }
    }

*Algorithm: one of the tasks of the Sobel filter.*

    task ComputeCrc {
      import com.synflow.net.mac.MacPack.*;
    
      in bool enable, sync u8 frame;
      out sync u8 frame_o;
    
      u32 crc_i;
      u3 i;
    
      void loop() {
        frame_o.write(frame.read);
        crc_i = 0xFFFF_FFFF;
    
        while (frame.available) {
          u8 data = frame.read();
          frame_o.write(data);
          bool en = enable.read;
          if (en) {
          crc_i = compute_crc(crc_i, data);
          }
        }
    
        // 3.2.8 Frame Check Sequence (FCS) field
        // CRC is transmitted from bit 31 to bit 0
        // see also 3.3 Order of bit transmission
        // FCS is transmitted high-order bit first
        crc_i = ~crc_i;
        frame_o.write(switch_nibbles_1G(crc_i >> 24));
        for (i = 3; i != 0; i--) {
          crc_i <> 24));
        }
      }
    }

*Protocol: one of the task of an Ethernet MAC.*

Cx combines first-class task and parallelism support with dependency injection and inheritance to make it easy to write composable hardware designs. For example, assuming you wanted to write a [Sobel filter](http://en.wikipedia.org/wiki/Sobel_operator), you could either:

    network SobelFilter extends ImageFilter {
      kernel = new SobelKernel();
    }

*Extend the generic image filter and override the kernel.*

    network SobelMain {
      f = new Filter({inject: 'SobelKernel'});
    }

*Inject the Sobel kernel inside the generic image filter.*

# Open Source Hardware

Cx has been created by Synflow. Yet, it is not a proprietary language but open source language licensed under Eclipse Public License (EPL). The language specification is available on-line, and the language can be extended to add custom semantics and custom source transformations. By the way, the 'x' of Cx stands for extra, Cx is a language that is kind of like C + extra features.

# Quality of Results

Compiling code from hardware languages to a configuration data to be loaded into a FPGA or to logic elements to create an ASIC is a fully mastered process. Hardware compilers (aka synthesizers) are especially effective, they are able to achieve all kinds of optimizations to deliver the best Quality of Result (QoR). What matters is to provide those synthesizers with a code that respects the right set of design rules.

This is why the Cx code is compiled into a beautiful, readable, synthesizable, vendor-neutral hardware (VHDL or Verilog) code. This has been a requirement since day one for us, that it remains possible to describe code in Cx that is competitive with hand-written VHDL or Verilog code.

    -------------------------------------------------------------------------------
    library ieee;
    use ieee.std_logic_1164.all;
    use ieee.numeric_std.all;
    use ieee.std_logic_unsigned.all;
    
    library std;
    use std.textio.all;
    
    library work;
    use work.Helper_functions.all;
    
    -------------------------------------------------------------------------------
    entity PhyMIIMI is
      port (
        clock : in std_logic;
        reset_n : in std_logic;
        readTransaction : in std_logic;
        readTransaction_send : in std_logic;
        mdi : in std_logic;
        phyAddr : in std_logic_vector(4 downto 0);
        regAddr : in std_logic_vector(4 downto 0);
        din : in std_logic_vector(15 downto 0);
        mdo : out std_logic;
        mdoEn : out std_logic;
        writeDone : out std_logic;
        writeDone_send : out std_logic;
        dout : out std_logic_vector(15 downto 0);
        dout_send : out std_logic);
    end PhyMIIMI;
    
    -------------------------------------------------------------------------------
    architecture rtl_PhyMIIMI of PhyMIIMI is
    
      constant LENGTH_PRE : unsigned(5 downto 0) := "100000";
      constant LENGTH_DATA : unsigned(5 downto 0) := "010000";
      signal doRead : std_logic;
      signal regad : unsigned(4 downto 0);
      signal phyad : unsigned(4 downto 0);
      signal data : unsigned(15 downto 0);
      signal i : unsigned(5 downto 0);
    
    
      type FSM_type is (s_start, s_send_st, s_send_st_1, s_send_op, s_send_op_1, s_send_phyad, s_issueRead, s_issueRead_1, s_issueRead_2, s_issueRead_3, s_issueRead_4);
      signal FSM : FSM_type;
    
    
    begin
    
      PhyMIIMI_execute : process (reset_n, clock) is
        variable local_data_4 : unsigned(15 downto 0);
      begin
        if reset_n = '0' then
          doRead <= '0';
          regad <= "00000";
          phyad <= "00000";
          data <= x"0000";
          i <= "000000";
          mdo <= '0';
          mdoEn <= '0';
          writeDone <= '0';
          writeDone_send <= '0';
          dout <= (others => '0');
          dout_send <= '0';
          FSM    <= s_start;
        --
        elsif rising_edge(clock) then
          writeDone_send <= '0';
          dout_send <= '0';
          --
          case FSM is
            when s_start =>
              if to_boolean(readTransaction_send and '1') then
                -- Body of start_a (line 63)
                doRead <= readTransaction;
                phyad <= unsigned(phyAddr);
                regad <= unsigned(regAddr);
                data <= unsigned(din);
                i <= "000000";
                FSM <= s_send_st;
              end if;
            
            when s_send_st =>
              if to_boolean(to_std_logic((i < LENGTH_PRE))) then
                -- Body of send_st_a (line 72)
                mdoEn <= '1';
                mdo <= '1';
                i <= resize((resize(i, 7) + "0000001"), 6);
                FSM <= s_send_st;
              elsif to_boolean(not (to_std_logic((i < LENGTH_PRE)))) then
                -- Body of send_st_b (line 78)
                mdo <= '0';
                FSM <= s_send_st_1;
              end if;
            
            when s_send_st_1 =>
              if to_boolean('1') then
                -- Body of send_st_1_a (line 79)
                mdo <= '1';
                FSM <= s_send_op;
              end if;
            
            when s_send_op =>
              if to_boolean('1') then
                -- Body of send_op_a (line 84)
                mdo <= doRead;
                FSM <= s_send_op_1;
              end if;
            
            when s_send_op_1 =>
              if to_boolean('1') then
                -- Body of send_op_1_a (line 85)
                mdo <= not (doRead);
                i <= "000000";
                FSM <= s_send_phyad;
              end if;
            
            when s_send_phyad =>
              if to_boolean(to_std_logic((i < "000101"))) then
                -- Body of send_phyad_a (line 91)
                mdo <= to_std_logic(((phyad and "10000") /= "00000"));
                phyad <= resize(shift_left(resize(phyad, 6), 1), 5);
                i <= resize((resize(i, 7) + "0000001"), 6);
                FSM <= s_send_phyad;
              elsif to_boolean(not (to_std_logic((i < "000101")))) then
                -- Body of send_phyad_b (line 98)
                mdo <= to_std_logic(((regad and "10000") /= "00000"));
                i <= "000000";
                FSM <= s_issueRead;
              end if;
            
            when s_issueRead =>
              if to_boolean(to_std_logic((i < "000100"))) then
                -- Body of issueRead_a (line 100)
                mdo <= to_std_logic(((resize(shift_left(resize(regad, 6), 1), 5) and "10000") /= "00000"));
                regad <= resize(shift_left(resize(regad, 6), 1), 5);
                i <= resize((resize(i, 7) + "0000001"), 6);
                FSM <= s_issueRead;
              elsif to_boolean((not (to_std_logic((i < "000100"))) and doRead)) then
                -- Body of issueRead_b (line 106)
                mdoEn <= '0';
                FSM <= s_issueRead_1;
              elsif to_boolean((not (to_std_logic((i < "000100"))) and not (doRead))) then
                -- Body of issueRead_c (line 108)
                mdo <= '1';
                FSM <= s_issueRead_3;
              end if;
            
            when s_issueRead_1 =>
              if to_boolean('1') then
                -- Body of issueRead_1_a (line 106)
                data <= x"0000";
                i <= "000000";
                FSM <= s_issueRead_2;
              end if;
            
            when s_issueRead_2 =>
              if to_boolean(to_std_logic((i < LENGTH_DATA))) then
                -- Body of issueRead_2_a (line 106)
                if (to_boolean(mdi)) then
                  local_data_4 := (resize(shift_left(resize(data, 17), 1), 16) or x"0001");
                else
                  local_data_4 := (resize(shift_left(resize(data, 17), 1), 16) and x"fffe");
                end if;
                data <= local_data_4;
                i <= resize((resize(i, 7) + "0000001"), 6);
                FSM <= s_issueRead_2;
              elsif to_boolean(not (to_std_logic((i < LENGTH_DATA)))) then
                -- Body of issueRead_2_b (line 106)
                dout <= std_logic_vector(data);
                dout_send <= '1';
                FSM <= s_start;
              end if;
            
            when s_issueRead_3 =>
              if to_boolean('1') then
                -- Body of issueRead_3_a (line 108)
                mdo <= '0';
                i <= "000000";
                FSM <= s_issueRead_4;
              end if;
            
            when s_issueRead_4 =>
              if to_boolean(to_std_logic((i < LENGTH_DATA))) then
                -- Body of issueRead_4_a (line 108)
                mdo <= to_std_logic(((data and x"8000") /= x"0000"));
                data <= resize(shift_left(resize(data, 17), 1), 16);
                i <= resize((resize(i, 7) + "0000001"), 6);
                FSM <= s_issueRead_4;
              elsif to_boolean(not (to_std_logic((i < LENGTH_DATA)))) then
                -- Body of issueRead_4_b (line 108)
                mdoEn <= '0';
                writeDone <= '1';
                writeDone_send <= '1';
                FSM <= s_start;
              end if;
          end case;
        end if;
      end process PhyMIIMI_execute;
    
    end architecture rtl_PhyMIIMI;

On top of that, if you ever want to walk away from Cx, you can keep working using the generated code. Note: we've removed the header comments from the example below so you can read code rather than a license text, but otherwise comments from the source are carried in the generated code, too.
