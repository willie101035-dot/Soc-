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
            slow_clk <= '0';
        elsif rising_edge(clk) then
            if div_counter = DIV_CNT - 1 then
                div_counter <= 0;
                slow_clk <= not slow_clk;
            else
                div_counter <= div_counter + 1;
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

    pwm_out <= '1' when (now_state = counter1 and pwm_cnt < Qn) else '0';

    process(slow_clk, rst)
    begin
        if rst = '1' then
            now_state <= counter1;
        elsif rising_edge(slow_clk) then
            now_state <= next_state;
        end if;
    end process;

    process(slow_clk, rst)
    begin
        if rst = '1' then
            Qn <= lower;
        elsif rising_edge(slow_clk) then
            Qn <= Qn_d;
        end if;

    end process;

    process(now_state, Qn)
    begin
        next_state <= now_state;

        case now_state is
            when counter1 =>
                if Qn >= upper then
                    next_state <= counter2;
                end if;

            when counter2 =>
                if Qn <= lower then
                    next_state <= counter1;
                end if;
        end case;
    end process;

    process(now_state, Qn)
    begin
        Qn_d <= Qn;

        case now_state is
            when counter1 =>
                if Qn >= upper then
                    Qn_d <= Qn - 1;
                else
                    Qn_d <= Qn + 1;
                end if;

            when counter2 =>
                if Qn <= lower then
                    Qn_d <= Qn + 1;
                else
                    Qn_d <= Qn - 1;
                end if;
        end case;
    end process;


    q <= std_logic_vector(to_unsigned(Qn, 4));

end Behavioral;
```
## 波形圖

## 架構圖

## break down
<img width="881" height="299" alt="image" src="https://github.com/user-attachments/assets/b248f968-8afd-4454-a75e-4343f8912879" />

## AOV

## fsm

## MSC

