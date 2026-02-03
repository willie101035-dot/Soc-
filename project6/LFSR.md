# LFSR
## 程式碼
```vhdl

library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity pingpong_top is
    port(
        clk          : in  std_logic;
        rst          : in  std_logic;
        btn_l        : in  std_logic;
        btn_r        : in  std_logic;
        led_out      : out std_logic_vector(7 downto 0)
    );
end pingpong_top;

architecture Behavioral of pingpong_top is
    constant DIV_CNT     : integer := 10000000;
    signal div_counter   : integer range 0 to DIV_CNT + 15 := 0;
    signal speed_limit   : integer range 0 to 15 := 0;
    signal lfsr_reg      : std_logic_vector(3 downto 0) := "1011";
    signal slow_clk      : std_logic := '0';
    signal ball_reg      : std_logic_vector(7 downto 0) := "10000000";
    signal dir           : std_logic := '0';
    signal score_l       : unsigned(3 downto 0) := "0000";
    signal score_r       : unsigned(3 downto 0) := "0000";
    type state_type is (PLAYING, SHOW_SCORE);
    signal current_state : state_type := PLAYING;
    signal delay_cnt     : integer range 0 to 10 := 0;
begin

    process(clk, rst)
    begin
        if rst = '1' then
            lfsr_reg <= "1011";
        elsif rising_edge(clk) then
            lfsr_reg <= lfsr_reg(2 downto 0) & (lfsr_reg(3) xor lfsr_reg(2));
        end if;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            speed_limit <= 0;
        elsif rising_edge(clk) then
            if (ball_reg = "00000010" and btn_r = '1') or 
               (ball_reg = "01000000" and btn_l = '1') or
               (current_state = SHOW_SCORE and delay_cnt = 10) then
                speed_limit <= to_integer(unsigned(lfsr_reg));
            end if;
        end if;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            div_counter <= 0;
        elsif rising_edge(clk) then
            if div_counter >= (DIV_CNT + (speed_limit * 100000)) then
                div_counter <= 0;
            else
                div_counter <= div_counter + 1;
            end if;
        end if;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            slow_clk <= '0';
        elsif rising_edge(clk) then
            if div_counter >= (DIV_CNT + (speed_limit * 100000)) then
                slow_clk <= not slow_clk;
            end if;
        end if;
    end process;

    process(slow_clk, rst)
    begin
        if rst = '1' then
            current_state <= PLAYING;
        elsif rising_edge(slow_clk) then
            case current_state is
                when PLAYING =>
                    if (dir = '0' and ball_reg = "00000001") or (dir = '1' and ball_reg = "10000000") then
                        current_state <= SHOW_SCORE;
                    end if;
                when SHOW_SCORE =>
                    if delay_cnt >= 10 then
                        current_state <= PLAYING;
                    end if;
            end case;
        end if;
    end process;

    process(slow_clk, rst)
    begin
        if rst = '1' then
            ball_reg <= "10000000";
        elsif rising_edge(slow_clk) then
            if current_state = SHOW_SCORE then
                if delay_cnt >= 10 then
                    if dir = '0' then
                        ball_reg <= "00000001";
                    else
                        ball_reg <= "10000000";
                    end if;
                end if;
            else
                if dir = '0' then
                    if ball_reg = "00000010" and btn_r = '1' then
                        ball_reg <= "00000100";
                    else
                        ball_reg <= '0' & ball_reg(7 downto 1);
                    end if;
                else
                    if ball_reg = "01000000" and btn_l = '1' then
                        ball_reg <= "00100000";
                    else
                        ball_reg <= ball_reg(6 downto 0) & '0';
                    end if;
                end if;
            end if;
        end if;
    end process;

    process(slow_clk, rst)
    begin
        if rst = '1' then
            dir <= '0';
        elsif rising_edge(slow_clk) then
            if current_state = SHOW_SCORE then
                if delay_cnt >= 10 then
                    dir <= not dir;
                end if;
            elsif dir = '0' and ball_reg = "00000010" and btn_r = '1' then
                dir <= '1';
            elsif dir = '1' and ball_reg = "01000000" and btn_l = '1' then
                dir <= '0';
            end if;
        end if;
    end process;

    process(slow_clk, rst)
    begin
        if rst = '1' then
            score_l <= (others => '0');
        elsif rising_edge(slow_clk) then
            if current_state = PLAYING and dir = '0' and ball_reg = "00000001" then
                score_l <= score_l + 1;
            end if;
        end if;
    end process;

    process(slow_clk, rst)
    begin
        if rst = '1' then
            score_r <= (others => '0');
        elsif rising_edge(slow_clk) then
            if current_state = PLAYING and dir = '1' and ball_reg = "10000000" then
                score_r <= score_r + 1;
            end if;
        end if;
    end process;

    process(slow_clk, rst)
    begin
        if rst = '1' then
            delay_cnt <= 0;
        elsif rising_edge(slow_clk) then
            if current_state = SHOW_SCORE then
                if delay_cnt < 10 then
                    delay_cnt <= delay_cnt + 1;
                else
                    delay_cnt <= 0;
                end if;
            else
                delay_cnt <= 0;
            end if;
        end if;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            led_out <= (others => '0');
        elsif rising_edge(clk) then
            if current_state = PLAYING then
                led_out <= ball_reg;
            else
                led_out <= std_logic_vector(score_l) & std_logic_vector(score_r);
            end if;
        end if;
    end process;

end Behavioral;

```
## 波形圖
<img width="1667" height="843" alt="image" src="https://github.com/user-attachments/assets/8487194d-6d23-4cb2-8b4c-d162862dfce0" />

## 架構圖

## break down
<img width="1862" height="352" alt="image" src="https://github.com/user-attachments/assets/6150d2b8-b9ea-404b-82ee-228a41d47fd8" />

## AOV

## fsm

## MSC

## FPGA板
https://github.com/user-attachments/assets/d5949197-7af8-4a6b-aef1-fefa74eb69f1


