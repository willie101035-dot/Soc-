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
    constant DIV_CNT     : integer := 2000000; 
    
    signal div_counter   : integer range 0 to DIV_CNT - 1 := 0;
    signal slow_clk      : std_logic := '0';
    
    signal ball_reg      : std_logic_vector(7 downto 0);
    signal dir           : std_logic; -- 0: To Right, 1: To Left
    signal score_l       : unsigned(3 downto 0);
    signal score_r       : unsigned(3 downto 0);
    
    type state_type is (WAIT_SERVE, PLAYING, SHOW_SCORE);
    signal current_state : state_type;
    signal delay_cnt     : integer range 0 to 10;
begin
    --div_counter
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
    --slow_clk
    process(clk, rst)
    begin
        if rst = '1' then
            slow_clk <= '0';
        elsif rising_edge(clk) then
            if div_counter = DIV_CNT - 1 then
                slow_clk <= '1'; -- 到了分頻點，拉高一個時脈週期
            else
                slow_clk <= '0'; -- 平時維持低電平
            end if;
        end if;
    end process;

    --current_state
    process(clk, rst)
    begin
        if rst = '1' then
            current_state <= WAIT_SERVE;
        elsif rising_edge(clk) then
            if slow_clk = '1' then
                case current_state is
                    when WAIT_SERVE =>
                        if (ball_reg(7) = '1' and btn_l = '1') or (ball_reg(0) = '1' and btn_r = '1') then
                            current_state <= PLAYING;
                        end if;
                    when PLAYING =>
                        -- 漏球，或者提早擊球（球沒進入安全區就按鍵），都切換到 SHOW_SCORE
                        if (dir = '0' and ball_reg(0) = '1') or (dir = '0' and ball_reg(1 downto 0) = "00" and btn_r = '1') then
                            current_state <= SHOW_SCORE;
                        elsif (dir = '1' and ball_reg(7) = '1') or (dir = '1' and ball_reg(7 downto 6) = "00" and btn_l = '1') then
                            current_state <= SHOW_SCORE;
                        end if;
                    when SHOW_SCORE =>
                        if delay_cnt = 10 then
                            current_state <= WAIT_SERVE;
                        end if;
                end case;
            end if;
        end if;
    end process;

--ball_reg
process(clk, rst)
begin
    if rst = '1' then
        ball_reg <= "10000000";
    elsif rising_edge(clk) then
        if slow_clk = '1' then
            if current_state = SHOW_SCORE and delay_cnt = 10 then
                null; 
            elsif current_state = PLAYING then
                if (dir = '0' and ball_reg(0) = '1') or (dir = '0' and ball_reg(1 downto 0) = "00" and btn_r = '1') then
                    ball_reg <= "00000001"; 
                elsif (dir = '1' and ball_reg(7) = '1') or (dir = '1' and ball_reg(7 downto 6) = "00" and btn_l = '1') then
                    ball_reg <= "10000000";
                elsif dir = '0' and ball_reg(1) = '1' and btn_r = '1' then
                    ball_reg <= ball_reg(6 downto 0) & '0';
                elsif dir = '1' and ball_reg(6) = '1' and btn_l = '1' then
                    ball_reg <= '0' & ball_reg(7 downto 1);
                elsif dir = '0' then
                    ball_reg <= '0' & ball_reg(7 downto 1);
                else
                    ball_reg <= ball_reg(6 downto 0) & '0';
                end if;
            end if;
        end if;
    end if;
end process;
    --dir
    process(clk, rst)
    begin
        if rst = '1' then
            dir <= '0';
        elsif rising_edge(clk) then
            if slow_clk = '1' then
                if current_state = WAIT_SERVE then
                    if ball_reg(7) = '1' and btn_l = '1' then dir <= '0';
                    elsif ball_reg(0) = '1' and btn_r = '1' then dir <= '1';
                    end if;
                elsif current_state = PLAYING then
                    if dir = '0' and ball_reg(1) = '1' and btn_r = '1' then dir <= '1';
                    elsif dir = '1' and ball_reg(6) = '1' and btn_l = '1' then dir <= '0';
                    end if;
                end if;
            end if;
        end if;
    end process;
    --score_l
    process(clk, rst)
    begin
        if rst = '1' then
            score_l <= (others => '0');
        elsif rising_edge(clk) then
            if slow_clk = '1' then
                if current_state = PLAYING then
                    -- 右邊漏球，或者右邊提早擊球，左邊都得 1 分
                    if (dir = '0' and ball_reg(0) = '1') or (dir = '0' and ball_reg(1 downto 0) = "00" and btn_r = '1') then
                        score_l <= score_l + 1;
                    end if;
                end if;
            end if;
        end if;
    end process;
    --score_r
    process(clk, rst)
    begin
        if rst = '1' then
            score_r <= (others => '0');
        elsif rising_edge(clk) then
            if slow_clk = '1' then
                if current_state = PLAYING then
                    if (dir = '1' and ball_reg(7) = '1') or (dir = '1' and ball_reg(7 downto 6) = "00" and btn_l = '1') then
                        score_r <= score_r + 1;
                    end if;
                end if;
            end if;
        end if;
    end process;
    --delay_cnt
    process(clk, rst)
    begin
        if rst = '1' then
            delay_cnt <= 0;
        elsif rising_edge(clk) then
            if slow_clk = '1' then
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
        end if;
    end process;
    --led_out
    process(clk, rst)
    begin
        if rst = '1' then
            led_out <= (others => '0');
        elsif rising_edge(clk) then
            if current_state = SHOW_SCORE then
                led_out <= std_logic_vector(score_l) & std_logic_vector(score_r);
            else
                led_out <= ball_reg;
            end if;
        end if;
    end process;

end Behavioral;
```
## 波形圖
<img width="1086" height="802" alt="image" src="https://github.com/user-attachments/assets/88dbfeba-e15f-4cdb-837a-683b8b8d2a38" />

## 架構圖
<img width="1889" height="723" alt="image" src="https://github.com/user-attachments/assets/bd7b520b-0939-4306-b193-4220ec21c444" />

## break down
<img width="1077" height="276" alt="image" src="https://github.com/user-attachments/assets/8a10929f-516f-440f-ac25-5443e35d7827" />

## AOV
<img width="1012" height="163" alt="image" src="https://github.com/user-attachments/assets/bbf6824a-b685-40ef-ba40-6bbcd5a596d6" />

## fsm
<img width="819" height="427" alt="image" src="https://github.com/user-attachments/assets/7e63a01d-e490-44bc-8fc1-5dc2b5b669e6" />

## MSC
<img width="1111" height="238" alt="image" src="https://github.com/user-attachments/assets/f162541f-6974-4b17-8dbb-6d0d4bd4cbfb" />


## FPGA板
https://github.com/user-attachments/assets/b4efa272-41de-47b1-87de-d062a1bd59b8




