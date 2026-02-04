# PWM
## 程式碼
```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;

entity PWM is
    generic(
        upper : integer := 9;
        lower : integer := 0;
        DIV_CNT : integer := 50000000 
    );
    port(
        rst      : in  STD_LOGIC;
        clk      : in  STD_LOGIC; 
        q        : out STD_LOGIC_VECTOR(3 downto 0);
        pwm_out  : out STD_LOGIC  
    );
end PWM;

architecture Behavioral of PWM is
    type counter_state is (counter1, counter2);
    signal now_state, next_state : counter_state;
    signal Qn, Qn_d : integer := lower;
    signal slow_clk : STD_LOGIC := '0';
    signal div_counter : integer range 0 to DIV_CNT := 0;
    signal pwm_cnt : integer range 0 to 9 := 0;
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

    process(clk, rst)
    begin
        if rst = '1' then
            pwm_cnt <= 0;
        elsif rising_edge(clk) then
            if pwm_cnt = 9 then
                pwm_cnt <= 0;
            else
                pwm_cnt <= pwm_cnt + 1;
            end if;
        end if;
    end process;

    process(slow_clk, rst)
    begin
        if rst = '1' then
            now_state <= counter1;
        elsif rising_edge(slow_clk) then
            now_state <= next_state;
        end if;
    end process;

    process(now_state, Qn, rst)
    begin
        if rst = '1' then
            next_state <= counter1;
        else
            case now_state is
                when counter1 =>
                    if Qn >= upper then
                        next_state <= counter2;
                    else
                        next_state <= counter1;
                    end if;
                when counter2 =>
                    if Qn <= lower then
                        next_state <= counter1;
                    else
                        next_state <= counter2;
                    end if;
            end case;
        end if;
    end process;

    process(slow_clk, rst)
    begin
        if rst = '1' then
            Qn <= lower;
        elsif rising_edge(slow_clk) then
            case now_state is
                when counter1 =>
                    if Qn < upper then
                        Qn <= Qn + 1;
                    end if;
                when counter2 =>
                    if Qn > lower then
                        Qn <= Qn - 1;
                    end if;
            end case;
        end if;
    end process;

    process(slow_clk, rst)
    begin
        if rst = '1' then
            Qn_d <= lower;
        elsif rising_edge(slow_clk) then
            Qn_d <= Qn;
        end if;
    end process;

    process(pwm_cnt, Qn, rst)
    begin
        if rst = '1' then
            pwm_out <= '0';
        elsif (pwm_cnt < Qn) then
            pwm_out <= '1';
        else
            pwm_out <= '0';
        end if;
    end process;

    q <= std_logic_vector(to_unsigned(Qn_d, 4));

end Behavioral;
```
## 波形圖
<img width="1123" height="585" alt="image" src="https://github.com/user-attachments/assets/a5eb9c68-bd89-45a1-b927-180653eb857f" />

## 架構圖
<img width="1746" height="603" alt="image" src="https://github.com/user-attachments/assets/ccb629fd-7482-4fd9-b6aa-f2fd5cd23397" />

## break down
<img width="1151" height="327" alt="image" src="https://github.com/user-attachments/assets/7060020d-855d-4e17-b667-287375b391d5" />

## AOV
<img width="1272" height="331" alt="image" src="https://github.com/user-attachments/assets/53626493-0fdc-4fb5-b59a-73f9dd5f603b" />

## FSM
<img width="461" height="499" alt="image" src="https://github.com/user-attachments/assets/ebc72dc5-99c7-4167-bc7d-0c7b054ea301" />

## MSC
<img width="1568" height="399" alt="image" src="https://github.com/user-attachments/assets/1a174127-1cdd-4286-85e5-7a9318c69381" />

## FPGA板
https://github.com/user-attachments/assets/4252a5e2-fb82-4c43-b9d9-31827bc85ec6



