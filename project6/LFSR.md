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
    constant DIV_CNT     : integer := 2000000;
    
    signal div_counter   : integer range 0 to 2000000 := 0; 
    signal speed_limit   : integer range 0 to 15 := 0;
    signal lfsr_reg      : std_logic_vector(3 downto 0) := "1011";
    signal slow_clk      : std_logic := '0';
    signal slow_clk_reg  : std_logic := '0'; 
    
    signal ball_reg      : std_logic_vector(9 downto 0) := "0100000000";
    signal dir           : std_logic := '0';
    signal score_l       : unsigned(3 downto 0) := "0000";
    signal score_r       : unsigned(3 downto 0) := "0000";
    
    type state_type is (WAIT_SERVE, PLAYING, SHOW_SCORE);
    signal current_state : state_type := WAIT_SERVE;
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
            div_counter <= 0;
        elsif rising_edge(clk) then
            if div_counter >= (DIV_CNT + speed_limit) then
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
            if div_counter >= (DIV_CNT + speed_limit) then
                slow_clk <= '1';
            else
                slow_clk <= '0';
            end if;
        end if;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            slow_clk_reg <= '0';
        elsif rising_edge(clk) then
            slow_clk_reg <= slow_clk;
        end if;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            speed_limit <= 0;
        elsif rising_edge(clk) then
            if (current_state = PLAYING and (ball_reg(1) = '1' or ball_reg(0) = '1') and btn_r = '1') or
               (current_state = PLAYING and (ball_reg(9) = '1' or ball_reg(8) = '1') and btn_l = '1') or
               (current_state = SHOW_SCORE and delay_cnt = 10) then
                speed_limit <= to_integer(unsigned(lfsr_reg));
            end if;
        end if;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            current_state <= WAIT_SERVE;
        elsif rising_edge(clk) then
            if slow_clk = '1' and slow_clk_reg = '0' then
                case current_state is
                    when WAIT_SERVE =>
                        if (ball_reg(8) = '1' and btn_l = '1') or (ball_reg(1) = '1' and btn_r = '1') then 
                            current_state <= PLAYING;
                        end if;
                        
                    when PLAYING =>
                        if dir = '0' and (ball_reg(1) = '1' or ball_reg(0) = '1') and btn_r = '1' then
                            current_state <= PLAYING;
                        elsif dir = '1' and (ball_reg(9) = '1' or ball_reg(8) = '1') and btn_l = '1' then
                            current_state <= PLAYING;
                        elsif (dir = '0' and ball_reg(0) = '1') or (dir = '0' and ball_reg(1) = '0' and ball_reg(0) = '0' and btn_r = '1') then
                            current_state <= SHOW_SCORE;
                        elsif (dir = '1' and ball_reg(9) = '1') or (dir = '1' and ball_reg(9) = '0' and ball_reg(8) = '0' and btn_l = '1') then
                            current_state <= SHOW_SCORE;
                        end if;
                        
                    when SHOW_SCORE =>
                        if delay_cnt >= 10 then
                            current_state <= WAIT_SERVE;
                        end if;
                end case;
            end if;
        end if;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            dir <= '0';
        elsif rising_edge(clk) then
            if slow_clk = '1' and slow_clk_reg = '0' then
                if current_state = WAIT_SERVE then
                    if (ball_reg(8) = '1' and btn_l = '1') then dir <= '0';
                    elsif (ball_reg(1) = '1' and btn_r = '1') then dir <= '1';
                    end if;
                elsif current_state = PLAYING then
                    if dir = '0' and (ball_reg(1) = '1' or ball_reg(0) = '1') and btn_r = '1' then dir <= '1';
                    elsif dir = '1' and (ball_reg(9) = '1' or ball_reg(8) = '1') and btn_l = '1' then dir <= '0';
                    end if;
                end if;
            end if;
        end if;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            ball_reg <= "0100000000";
        elsif rising_edge(clk) then
            if slow_clk = '1' and slow_clk_reg = '0' then
                case current_state is
                    when WAIT_SERVE =>
                        null;
                        
                    when PLAYING =>
                        if dir = '0' and (ball_reg(1) = '1' or ball_reg(0) = '1') and btn_r = '1' then 
                            ball_reg <= ball_reg(8 downto 0) & '0';
                        elsif dir = '1' and (ball_reg(9) = '1' or ball_reg(8) = '1') and btn_l = '1' then 
                            ball_reg <= '0' & ball_reg(9 downto 1);
                        elsif (dir = '0' and ball_reg(0) = '1') or (dir = '0' and ball_reg(1) = '0' and ball_reg(0) = '0' and btn_r = '1') then 
                            ball_reg <= "0000000010"; 
                        elsif (dir = '1' and ball_reg(9) = '1') or (dir = '1' and ball_reg(9) = '0' and ball_reg(8) = '0' and btn_l = '1') then 
                            ball_reg <= "0100000000";
                        elsif dir = '0' then 
                            ball_reg <= '0' & ball_reg(9 downto 1);
                        else 
                            ball_reg <= ball_reg(8 downto 0) & '0';
                        end if;
                        
                    when SHOW_SCORE =>
                        if delay_cnt >= 10 then
                            if score_l > score_r then
                                ball_reg <= "0100000000";
                            else
                                ball_reg <= "0000000010";
                            end if;
                        end if;
                end case;
            end if;
        end if;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            score_l <= "0000";
        elsif rising_edge(clk) then
            if slow_clk = '1' and slow_clk_reg = '0' then
                if current_state = PLAYING then
                    if dir = '0' and (ball_reg(1) = '1' or ball_reg(0) = '1') and btn_r = '1' then
                        score_l <= score_l;
                    elsif (dir = '0' and ball_reg(0) = '1') or (dir = '0' and ball_reg(1) = '0' and ball_reg(0) = '0' and btn_r = '1') then
                        score_l <= score_l + 1;
                    end if;
                end if;
            end if;
        end if;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            score_r <= "0000";
        elsif rising_edge(clk) then
            if slow_clk = '1' and slow_clk_reg = '0' then
                if current_state = PLAYING then
                    if dir = '1' and (ball_reg(9) = '1' or ball_reg(8) = '1') and btn_l = '1' then
                        score_r <= score_r;
                    elsif (dir = '1' and ball_reg(9) = '1') or (dir = '1' and ball_reg(9) = '0' and ball_reg(8) = '0' and btn_l = '1') then
                        score_r <= score_r + 1;
                    end if;
                end if;
            end if;
        end if;
    end process;

    process(clk, rst)
    begin
        if rst = '1' then
            delay_cnt <= 0;
        elsif rising_edge(clk) then
            if slow_clk = '1' and slow_clk_reg = '0' then
                if current_state = SHOW_SCORE then
                    if delay_cnt >= 10 then
                        delay_cnt <= 0;
                    else
                        delay_cnt <= delay_cnt + 1;
                    end if;
                else
                    delay_cnt <= 0;
                end if;
            end if;
        end if;
    end process;

    led_out <= std_logic_vector(score_l) & std_logic_vector(score_r) when current_state = SHOW_SCORE else ball_reg(8 downto 1);

end Behavioral;

```
## 波形圖
<img width="1546" height="787" alt="image" src="https://github.com/user-attachments/assets/01f6c315-6461-41be-8859-380f781d2a13" />

## 架構圖
<img width="935" height="453" alt="image" src="https://github.com/user-attachments/assets/a53b25a9-6b5d-478c-8a8c-d0613458411d" />

## break down
<img width="1862" height="352" alt="image" src="https://github.com/user-attachments/assets/6150d2b8-b9ea-404b-82ee-228a41d47fd8" />

## AOV
<img width="1012" height="163" alt="image" src="https://github.com/user-attachments/assets/bbf6824a-b685-40ef-ba40-6bbcd5a596d6" />
    
## fsm
<img width="819" height="427" alt="image" src="https://github.com/user-attachments/assets/7e63a01d-e490-44bc-8fc1-5dc2b5b669e6" />

## MSC
<img width="1656" height="145" alt="image" src="https://github.com/user-attachments/assets/bbdbc45f-83ed-4b0c-8145-d182d88d7f74" />

## FPGA板
https://github.com/user-attachments/assets/fa0058a8-c8ca-488a-9778-fb65c654555f



