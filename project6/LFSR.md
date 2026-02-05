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
    signal div_counter   : integer range 0 to DIV_CNT + 1500000 := 0; -- 修正範圍
    signal speed_limit   : integer range 0 to 15 := 0;
    signal lfsr_reg      : std_logic_vector(3 downto 0) := "1011";
    signal slow_clk      : std_logic := '0';
    signal ball_reg      : std_logic_vector(7 downto 0) := "10000000";
    signal dir           : std_logic := '0';
    signal score_l       : unsigned(3 downto 0) := "0000";
    signal score_r       : unsigned(3 downto 0) := "0000";
    
    -- 新增 WAIT_SERVE 狀態
    type state_type is (WAIT_SERVE, PLAYING, SHOW_SCORE);
    signal current_state : state_type := WAIT_SERVE;
    signal delay_cnt     : integer range 0 to 10 := 0;
begin

    -- 1. LFSR 隨機數
    process(clk, rst)
    begin
        if rst = '1' then
            lfsr_reg <= "1011";
        elsif rising_edge(clk) then
            lfsr_reg <= lfsr_reg(2 downto 0) & (lfsr_reg(3) xor lfsr_reg(2));
        end if;
    end process;

    -- 2. 隨機速度界限 (球被擊中或得分後更新)
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

    -- 3. 分頻計數器
    process(clk, rst)
    begin
        if rst = '1' then
            div_counter <= 0;
        elsif rising_edge(clk) then
            -- 這裡乘上 100000 讓隨機感明顯
            if div_counter >= (DIV_CNT + (speed_limit * 100000)) then
                div_counter <= 0;
            else
                div_counter <= div_counter + 1;
            end if;
        end if;
    end process;

    -- 4. 慢速時鐘
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

    -- 5. 狀態機 (加入 WAIT_SERVE)
    process(slow_clk, rst)
    begin
        if rst = '1' then
            current_state <= WAIT_SERVE;
        elsif rising_edge(slow_clk) then
            case current_state is
                when WAIT_SERVE =>
                    -- 手動發球判定
                    if (ball_reg(7) = '1' and btn_l = '1') or (ball_reg(0) = '1' and btn_r = '1') then
                        current_state <= PLAYING;
                    end if;
                when PLAYING =>
                    if (dir = '0' and ball_reg = "00000001") or (dir = '1' and ball_reg = "10000000") then
                        current_state <= SHOW_SCORE;
                    end if;
                when SHOW_SCORE =>
                    if delay_cnt >= 10 then
                        current_state <= WAIT_SERVE;
                    end if;
            end case;
        end if;
    end process;

    -- 6. 球的位置 (PLAYING 時才移動)
    process(slow_clk, rst)
    begin
        if rst = '1' then
            ball_reg <= "10000000";
        elsif rising_edge(slow_clk) then
            if current_state = SHOW_SCORE and delay_cnt >= 10 then
                -- 準備發球位置
                if dir = '0' then ball_reg <= "00000001"; else ball_reg <= "10000000"; end if;
            elsif current_state = PLAYING then
                if dir = '0' then
                    if ball_reg = "00000010" and btn_r = '1' then ball_reg <= "00000100";
                    else ball_reg <= '0' & ball_reg(7 downto 1); end if;
                else
                    if ball_reg = "01000000" and btn_l = '1' then ball_reg <= "00100000";
                    else ball_reg <= ball_reg(6 downto 0) & '0'; end if;
                end if;
            end if;
        end if;
    end process;

    -- 7. 移動方向
    process(slow_clk, rst)
    begin
        if rst = '1' then
            dir <= '0';
        elsif rising_edge(slow_clk) then
            if current_state = WAIT_SERVE then
                if ball_reg(7) = '1' and btn_l = '1' then dir <= '0';
                elsif ball_reg(0) = '1' and btn_r = '1' then dir <= '1'; end if;
            elsif current_state = PLAYING then
                if dir = '0' and ball_reg = "00000010" and btn_r = '1' then dir <= '1';
                elsif dir = '1' and ball_reg = "01000000" and btn_l = '1' then dir <= '0'; end if;
            end if;
        end if;
    end process;

    -- 8. 左得分
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

    -- 9. 右得分
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

    -- 10. 延遲計數
    process(slow_clk, rst)
    begin
        if rst = '1' then
            delay_cnt <= 0;
        elsif rising_edge(slow_clk) then
            if current_state = SHOW_SCORE then
                if delay_cnt < 10 then delay_cnt <= delay_cnt + 1; else delay_cnt <= 0; end if;
            else
                delay_cnt <= 0;
            end if;
        end if;
    end process;

    -- 11. LED 輸出
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
<img width="1539" height="778" alt="image" src="https://github.com/user-attachments/assets/2cc5cf5c-29b2-4cc0-966b-6b06c947cedd" />

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
https://github.com/user-attachments/assets/d5949197-7af8-4a6b-aef1-fefa74eb69f1


