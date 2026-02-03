# BALL
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
    constant DIV_CNT     : integer := 10;
    signal div_counter   : integer range 0 to DIV_CNT - 1 := 0;
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
            div_counter <= 0;
        elsif rising_edge(clk) then
            if div_counter = DIV_CNT - 1 then
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
            if div_counter = DIV_CNT - 1 then
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
                    if dir = '0' then -- 剛剛是右邊漏球
                        ball_reg <= "00000001"; -- 從右邊發球
                    else -- 剛剛是左邊漏球
                        ball_reg <= "10000000"; -- 從左邊發球
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
                    if dir = '0' then
                        dir <= '1'; 
                    else
                        dir <= '0'; 
                    end if;
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
<img width="938" height="617" alt="image" src="https://github.com/user-attachments/assets/c12dc11e-bf50-4ede-8627-aec348b0b97e" />

## 架構圖
<img width="1889" height="723" alt="image" src="https://github.com/user-attachments/assets/bd7b520b-0939-4306-b193-4220ec21c444" />

## break down
<img width="1077" height="276" alt="image" src="https://github.com/user-attachments/assets/8a10929f-516f-440f-ac25-5443e35d7827" />

## AOV
<img width="978" height="389" alt="image" src="https://github.com/user-attachments/assets/e484e99a-1dff-4493-b335-8d8cb75be5e2" />

## fsm
<img width="1080" height="671" alt="image" src="https://github.com/user-attachments/assets/74814f7a-fafa-4ebb-acad-9bc92e3e4a6e" />

## MSC

## FPGA板
https://github.com/user-attachments/assets/28c1f027-e03b-4392-b515-b896a25c505d




